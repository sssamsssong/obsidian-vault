# 모델 배포

> YAX F1TENTH 교육자료 | Phase 5: Data & ML
> 
> Jetson에서 모델 추론 및 ROS2 통합

---

## 1. 개요

### 학습 목표
1. 학습된 모델을 TensorRT로 최적화할 수 있다
2. Jetson에서 실시간 추론을 수행할 수 있다
3. ROS2 노드로 모델을 통합할 수 있다

### 예상 소요 시간
- 전체: 4시간

### 전제 조건
- E2E 모델 학습 완료
- Jetson 환경 설정 완료
- ROS2 기초

---

## 2. 배포 파이프라인

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   PyTorch    │────►│    ONNX      │────►│  TensorRT    │────►│   Jetson     │
│   (.pth)     │     │   (.onnx)    │     │   (.engine)  │     │   추론       │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
     학습               중간 형식            최적화                 실시간
```

---

## 3. ONNX 변환

### 3.1 PyTorch → ONNX

```python
import torch
import torch.onnx

def export_to_onnx(model_path: str, 
                   output_path: str,
                   input_size: tuple = (1, 3, 66, 200)):
    """
    PyTorch 모델을 ONNX로 변환.
    
    Args:
        model_path: PyTorch 모델 파일 경로
        output_path: ONNX 출력 경로
        input_size: 입력 텐서 크기 (batch, channels, height, width)
    """
    # 모델 로드
    model = PilotNet()  # 모델 클래스
    model.load_state_dict(torch.load(model_path, map_location='cpu'))
    model.eval()
    
    # 더미 입력
    dummy_input = torch.randn(*input_size)
    
    # ONNX 변환
    torch.onnx.export(
        model,
        dummy_input,
        output_path,
        input_names=['input'],
        output_names=['steering'],
        dynamic_axes={
            'input': {0: 'batch_size'},
            'steering': {0: 'batch_size'}
        },
        opset_version=11,
        do_constant_folding=True
    )
    
    print(f"ONNX model saved to: {output_path}")
    
    # 검증
    import onnx
    model_onnx = onnx.load(output_path)
    onnx.checker.check_model(model_onnx)
    print("ONNX model is valid!")


# 사용
export_to_onnx('best_model.pth', 'model.onnx')
```

### 3.2 ONNX 추론 테스트

```python
import onnxruntime as ort
import numpy as np

def test_onnx_inference(onnx_path: str):
    """ONNX 모델 추론 테스트."""
    
    # 세션 생성
    session = ort.InferenceSession(onnx_path)
    
    # 입력 정보
    input_name = session.get_inputs()[0].name
    input_shape = session.get_inputs()[0].shape
    print(f"Input: {input_name}, Shape: {input_shape}")
    
    # 출력 정보
    output_name = session.get_outputs()[0].name
    print(f"Output: {output_name}")
    
    # 테스트 추론
    dummy_input = np.random.randn(1, 3, 66, 200).astype(np.float32)
    result = session.run([output_name], {input_name: dummy_input})
    
    print(f"Output shape: {result[0].shape}")
    print(f"Output value: {result[0]}")
    
    return session
```

---

## 4. TensorRT 최적화

### 4.1 TensorRT 변환 (Jetson에서 실행)

```python
import tensorrt as trt

def build_tensorrt_engine(onnx_path: str,
                          engine_path: str,
                          fp16: bool = True,
                          max_batch_size: int = 1):
    """
    ONNX → TensorRT 엔진 변환.
    
    Jetson에서 실행해야 함!
    """
    logger = trt.Logger(trt.Logger.WARNING)
    builder = trt.Builder(logger)
    
    # 네트워크 정의
    network = builder.create_network(
        1 << int(trt.NetworkDefinitionCreationFlag.EXPLICIT_BATCH)
    )
    
    # ONNX 파서
    parser = trt.OnnxParser(network, logger)
    
    with open(onnx_path, 'rb') as f:
        if not parser.parse(f.read()):
            for error in range(parser.num_errors):
                print(parser.get_error(error))
            raise RuntimeError("ONNX parsing failed!")
    
    # 빌더 설정
    config = builder.create_builder_config()
    config.max_workspace_size = 1 << 30  # 1GB
    
    # FP16 최적화 (Jetson에서 빠름)
    if fp16 and builder.platform_has_fast_fp16:
        config.set_flag(trt.BuilderFlag.FP16)
        print("Using FP16 mode")
    
    # 최적화 프로파일
    profile = builder.create_optimization_profile()
    profile.set_shape(
        'input',
        min=(1, 3, 66, 200),
        opt=(1, 3, 66, 200),
        max=(max_batch_size, 3, 66, 200)
    )
    config.add_optimization_profile(profile)
    
    # 엔진 빌드 (시간 소요)
    print("Building TensorRT engine... (this may take a few minutes)")
    engine = builder.build_engine(network, config)
    
    if engine is None:
        raise RuntimeError("Engine build failed!")
    
    # 저장
    with open(engine_path, 'wb') as f:
        f.write(engine.serialize())
    
    print(f"TensorRT engine saved to: {engine_path}")
    return engine


# Jetson에서 실행
build_tensorrt_engine('model.onnx', 'model.engine', fp16=True)
```

### 4.2 trtexec 사용 (명령줄)

```bash
# ONNX → TensorRT (Jetson에서)
/usr/src/tensorrt/bin/trtexec \
    --onnx=model.onnx \
    --saveEngine=model.engine \
    --fp16 \
    --workspace=1024

# 성능 테스트
/usr/src/tensorrt/bin/trtexec \
    --loadEngine=model.engine \
    --iterations=100 \
    --warmUp=10
```

---

## 5. TensorRT 추론

### 5.1 추론 클래스

```python
import tensorrt as trt
import pycuda.driver as cuda
import pycuda.autoinit
import numpy as np


class TensorRTInference:
    """TensorRT 추론 클래스."""
    
    def __init__(self, engine_path: str):
        """
        Args:
            engine_path: TensorRT 엔진 파일 경로
        """
        self.logger = trt.Logger(trt.Logger.WARNING)
        
        # 엔진 로드
        with open(engine_path, 'rb') as f:
            runtime = trt.Runtime(self.logger)
            self.engine = runtime.deserialize_cuda_engine(f.read())
        
        self.context = self.engine.create_execution_context()
        
        # 입출력 바인딩
        self.input_shape = self.engine.get_binding_shape(0)
        self.output_shape = self.engine.get_binding_shape(1)
        
        # 메모리 할당
        self.d_input = cuda.mem_alloc(
            int(np.prod(self.input_shape) * np.float32().nbytes)
        )
        self.d_output = cuda.mem_alloc(
            int(np.prod(self.output_shape) * np.float32().nbytes)
        )
        
        self.bindings = [int(self.d_input), int(self.d_output)]
        self.stream = cuda.Stream()
        
        print(f"Loaded TensorRT engine: {engine_path}")
        print(f"  Input shape: {self.input_shape}")
        print(f"  Output shape: {self.output_shape}")
    
    def infer(self, input_data: np.ndarray) -> np.ndarray:
        """
        추론 실행.
        
        Args:
            input_data: (1, 3, 66, 200) numpy 배열
            
        Returns:
            (1, 1) 조향각 예측
        """
        # 입력 데이터 전송
        cuda.memcpy_htod_async(
            self.d_input, 
            input_data.astype(np.float32).ravel(),
            self.stream
        )
        
        # 추론 실행
        self.context.execute_async_v2(
            bindings=self.bindings,
            stream_handle=self.stream.handle
        )
        
        # 출력 가져오기
        output = np.empty(self.output_shape, dtype=np.float32)
        cuda.memcpy_dtoh_async(output, self.d_output, self.stream)
        
        self.stream.synchronize()
        
        return output
    
    def __del__(self):
        """리소스 해제."""
        if hasattr(self, 'd_input'):
            self.d_input.free()
        if hasattr(self, 'd_output'):
            self.d_output.free()


# 사용
inference = TensorRTInference('model.engine')

# 테스트
dummy = np.random.randn(1, 3, 66, 200).astype(np.float32)
result = inference.infer(dummy)
print(f"Prediction: {result}")
```

### 5.2 성능 측정

```python
import time

def benchmark_inference(inference, n_iterations: int = 100):
    """추론 성능 측정."""
    
    dummy = np.random.randn(1, 3, 66, 200).astype(np.float32)
    
    # 워밍업
    for _ in range(10):
        inference.infer(dummy)
    
    # 측정
    times = []
    for _ in range(n_iterations):
        start = time.perf_counter()
        inference.infer(dummy)
        end = time.perf_counter()
        times.append(end - start)
    
    times = np.array(times) * 1000  # ms
    
    print(f"Inference time:")
    print(f"  Mean: {np.mean(times):.2f} ms")
    print(f"  Std: {np.std(times):.2f} ms")
    print(f"  Min: {np.min(times):.2f} ms")
    print(f"  Max: {np.max(times):.2f} ms")
    print(f"  FPS: {1000 / np.mean(times):.1f}")


# 벤치마크
benchmark_inference(inference)
```

---

## 6. ROS2 추론 노드

### 6.1 E2E 추론 노드

```python
#!/usr/bin/env python3
"""
End-to-End 자율주행 추론 노드

카메라 이미지 → CNN → 조향각
"""

import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image, CompressedImage
from ackermann_msgs.msg import AckermannDriveStamped
from cv_bridge import CvBridge
import numpy as np
import cv2


class E2EInferenceNode(Node):
    def __init__(self):
        super().__init__('e2e_inference')
        
        # 파라미터
        self.declare_parameter('engine_path', 'model.engine')
        self.declare_parameter('image_topic', '/camera/image_raw/compressed')
        self.declare_parameter('speed', 2.0)
        self.declare_parameter('max_steering', 0.4)
        
        engine_path = self.get_parameter('engine_path').value
        image_topic = self.get_parameter('image_topic').value
        self.speed = self.get_parameter('speed').value
        self.max_steering = self.get_parameter('max_steering').value
        
        # TensorRT 추론 엔진
        self.inference = TensorRTInference(engine_path)
        
        # CV Bridge
        self.bridge = CvBridge()
        
        # 전처리 파라미터
        self.input_size = (200, 66)
        self.mean = np.array([0.485, 0.456, 0.406])
        self.std = np.array([0.229, 0.224, 0.225])
        
        # Subscriber
        if 'compressed' in image_topic:
            self.image_sub = self.create_subscription(
                CompressedImage,
                image_topic,
                self.compressed_image_callback,
                10
            )
        else:
            self.image_sub = self.create_subscription(
                Image,
                image_topic,
                self.image_callback,
                10
            )
        
        # Publisher
        self.drive_pub = self.create_publisher(
            AckermannDriveStamped,
            '/drive',
            10
        )
        
        self.get_logger().info(f'E2E Inference node started')
        self.get_logger().info(f'  Engine: {engine_path}')
        self.get_logger().info(f'  Image topic: {image_topic}')
    
    def preprocess(self, image: np.ndarray) -> np.ndarray:
        """
        이미지 전처리.
        
        Args:
            image: BGR 이미지 (H, W, 3)
            
        Returns:
            (1, 3, 66, 200) 정규화된 텐서
        """
        # BGR → RGB
        image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
        
        # ROI (상단 40%, 하단 10% 제거)
        h, w = image.shape[:2]
        image = image[int(h*0.4):int(h*0.9), :, :]
        
        # 리사이즈
        image = cv2.resize(image, self.input_size)
        
        # 정규화
        image = image.astype(np.float32) / 255.0
        image = (image - self.mean) / self.std
        
        # CHW 형식으로 변환
        image = image.transpose(2, 0, 1)
        
        # 배치 차원 추가
        image = np.expand_dims(image, axis=0)
        
        return image.astype(np.float32)
    
    def image_callback(self, msg: Image):
        """원본 이미지 콜백."""
        try:
            cv_image = self.bridge.imgmsg_to_cv2(msg, 'bgr8')
            self.process_image(cv_image)
        except Exception as e:
            self.get_logger().error(f'Image processing error: {e}')
    
    def compressed_image_callback(self, msg: CompressedImage):
        """압축 이미지 콜백."""
        try:
            np_arr = np.frombuffer(msg.data, np.uint8)
            cv_image = cv2.imdecode(np_arr, cv2.IMREAD_COLOR)
            self.process_image(cv_image)
        except Exception as e:
            self.get_logger().error(f'Image processing error: {e}')
    
    def process_image(self, image: np.ndarray):
        """이미지 처리 및 조향 명령 발행."""
        
        # 전처리
        input_data = self.preprocess(image)
        
        # 추론
        output = self.inference.infer(input_data)
        steering = float(output[0, 0])
        
        # 조향각 제한
        steering = np.clip(steering, -self.max_steering, self.max_steering)
        
        # 속도 조절 (조향각에 따라 감속)
        speed = self.speed * (1.0 - abs(steering) / self.max_steering * 0.3)
        
        # 명령 발행
        drive_msg = AckermannDriveStamped()
        drive_msg.header.stamp = self.get_clock().now().to_msg()
        drive_msg.header.frame_id = 'base_link'
        drive_msg.drive.speed = float(speed)
        drive_msg.drive.steering_angle = steering
        
        self.drive_pub.publish(drive_msg)
        
        self.get_logger().debug(f'Steering: {steering:.3f}, Speed: {speed:.2f}')


def main(args=None):
    rclpy.init(args=args)
    node = E2EInferenceNode()
    
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        rclpy.shutdown()


if __name__ == '__main__':
    main()
```

### 6.2 Launch 파일

```python
# launch/e2e_inference_launch.py
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(
            package='f1tenth_ml',
            executable='e2e_inference',
            name='e2e_inference',
            parameters=[{
                'engine_path': '/home/yax/models/model.engine',
                'image_topic': '/camera/image_raw/compressed',
                'speed': 2.0,
                'max_steering': 0.4,
            }],
            output='screen'
        )
    ])
```

---

## 7. 배포 워크플로우

### 7.1 PC에서 학습

```bash
# 1. 데이터 수집 (Jetson)
ros2 bag record -o driving_data /camera/image_raw/compressed /scan /drive

# 2. 데이터 전송
scp -r jetson@192.168.1.100:~/driving_data ./data/

# 3. 데이터 처리 (PC)
python bag_extractor.py data/driving_data ./processed_data

# 4. 학습 (PC with GPU)
python train.py --config configs/pilotnet.yaml

# 5. ONNX 변환 (PC)
python export_onnx.py best_model.pth model.onnx
```

### 7.2 Jetson에 배포

```bash
# 1. ONNX 전송
scp model.onnx jetson@192.168.1.100:~/models/

# 2. TensorRT 변환 (Jetson에서!)
ssh jetson@192.168.1.100
cd ~/models
python build_tensorrt.py model.onnx model.engine

# 3. 추론 노드 실행
ros2 launch f1tenth_ml e2e_inference_launch.py
```

---

## 8. 성능 최적화

### 8.1 Jetson 최적화

```bash
# 최대 성능 모드
sudo nvpmodel -m 0
sudo jetson_clocks

# 현재 모드 확인
sudo nvpmodel -q
```

### 8.2 추론 최적화 팁

| 방법 | 효과 |
|------|------|
| FP16 모드 | 2배 속도 향상 |
| 배치 크기 1 유지 | 지연 시간 최소화 |
| 비동기 추론 | CPU 활용 개선 |
| 이미지 전처리 최적화 | 병목 제거 |

### 8.3 메모리 최적화

```python
# 엔진 빌드 시 workspace 제한
config.max_workspace_size = 512 * 1024 * 1024  # 512MB

# INT8 양자화 (데이터 필요)
config.set_flag(trt.BuilderFlag.INT8)
```

---

## 9. 문제 해결

### 자주 발생하는 문제

| 증상 | 원인 | 해결책 |
|------|------|--------|
| ONNX 변환 실패 | 지원 안 되는 연산 | opset 버전 변경 |
| TensorRT 빌드 실패 | 메모리 부족 | workspace 줄이기 |
| 추론 결과 다름 | 전처리 불일치 | 동일 전처리 확인 |
| 느린 추론 | FP32 모드 | FP16 활성화 |
| OOM 에러 | GPU 메모리 부족 | 배치 크기 줄이기 |

### 디버깅

```bash
# TensorRT 로그 확인
export TRT_LOGGER_VERBOSE=1

# GPU 메모리 확인
nvidia-smi

# Jetson 상태 확인
tegrastats
```

---

## 10. 실습

### 실습 1: ONNX 변환

1. 학습된 모델을 ONNX로 변환
2. ONNX Runtime으로 추론 테스트

### 실습 2: TensorRT 변환 (Jetson)

1. ONNX → TensorRT 엔진 빌드
2. 추론 벤치마크

### 실습 3: ROS2 통합

1. E2E 추론 노드 실행
2. 시뮬레이터에서 테스트
3. 실차 테스트

---

## 11. 검증 체크리스트

- [ ] ONNX 변환 성공
- [ ] TensorRT 엔진 빌드 (Jetson)
- [ ] FP16 추론 확인
- [ ] ROS2 노드 동작
- [ ] 실시간 추론 (30+ FPS)
- [ ] 실차 주행 테스트

---

*실시간 추론은 자율주행의 핵심입니다. 최적화를 통해 안정적인 성능을 확보하세요!*
