# 분류 평가 지표 (Evaluation Metrics)

#분류 #평가지표

## 혼동 행렬 (Confusion Matrix)

|  | 예측 Positive | 예측 Negative |
|--|--------------|--------------|
| **실제 Positive** | TP (True Positive) | FN (False Negative) |
| **실제 Negative** | FP (False Positive) | TN (True Negative) |

---

## Accuracy (정확도)

$$\text{Accuracy} = \frac{TP + TN}{TP + TN + FP + FN}$$

- 전체 예측 중 맞은 비율
- **클래스 불균형 시 misleading** — 99% 음성 데이터에서 전부 음성 예측해도 99% 달성

---

## Precision (정밀도)

$$\text{Precision} = \frac{TP}{TP + FP}$$

- 모델이 Positive로 예측한 것 중 실제로 Positive인 비율
- **"내가 맞다고 한 것들 중 실제로 맞은 비율"**
- 중요한 경우: 스팸 분류 (정상 메일을 스팸으로 잘못 분류하면 안 됨)

---

## Recall (재현율)

$$\text{Recall} = \frac{TP}{TP + FN}$$

- 실제 Positive 중 모델이 Positive로 맞게 예측한 비율
- **"실제 양성을 얼마나 놓치지 않고 잡았나"**
- 중요한 경우: 암 진단 (실제 환자를 놓치면 안 됨)

---

## Precision-Recall Tradeoff

Precision이 높아지면 Recall은 낮아짐 (반대도 성립).

```
Threshold 낮춤 → 더 많이 Positive 예측 → Recall ↑, Precision ↓
Threshold 높임 → 더 적게 Positive 예측 → Precision ↑, Recall ↓
```

---

## F1-Score

Precision과 Recall의 조화 평균. 클래스 불균형 환경에서 필수.

$$F_1 = 2 \cdot \frac{\text{Precision} \times \text{Recall}}{\text{Precision} + \text{Recall}}$$

- 둘 다 높아야 F1도 높음
- 어느 한쪽만 높으면 F1은 낮게 유지

---

## 언제 무엇을 쓸까

| 상황 | 추천 지표 |
|------|-----------|
| 클래스 균형, 전반적 성능 | Accuracy |
| FP 비용이 높을 때 (스팸 필터) | Precision |
| FN 비용이 높을 때 (의료 진단) | Recall |
| 클래스 불균형, 종합 평가 | F1-Score |

## 연결 개념

- [[glue_tasks]] — GLUE 벤치마크도 이 지표로 평가
- [[sentiment_analysis]] — 감성 분류 성능 측정에 사용
