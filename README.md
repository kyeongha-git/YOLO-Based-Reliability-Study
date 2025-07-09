# 📘 Introduction

본 리포지토리는 2024년 한국정보과학회에 투고한  
**『운전자 안전을 위한 차량 유리 복원 교체 판단 모델 구축에서 과적합 가능성을 가진 데이터로부터 YOLO 기반 신뢰성 확보 연구』** 논문과  
이후 공개 예정인  
**『객체 탐지 기반 전처리를 활용한 차량 유리 손상 분류 신뢰도 향상 방안』** 논문 관련 **코드 및 결과**를 포함합니다.

---

# 📌 Description

## 📍 Background

국토교통부 통계에 따르면, **2023년 기준 국내 등록 차량은 2014년 대비 약 500만 대 증가**하였습니다.  
이는 **도로의 복잡성과 사고 발생률 증가**로 이어지며, **사고 이후의 대응 방식** 또한 점차 중요해지고 있습니다.

🚗 자동차 수리 시 **과도한 수리 비용**이 청구되는 경우, 많은 운전자가  
> "너무 많이 나온 것 같은데... 내가 몰라서 뭐라고 할 수가 없네..."  
와 같은 경험을 하곤 합니다.

🧠 이를 해결하기 위해, **AI를 통해 손상 수준과 수리 비용을 사전에 판단**할 수 있다면  
이러한 불합리한 상황을 줄이고, **정당한 수리비 청구 문화를 조성**할 수 있습니다.

이에 본 프로젝트에서는 **(주)에스에이시스템 제공 차량 유리 이미지 데이터**를 바탕으로,  
**AI 기반 손상 수준 자동 판별 아키텍처**를 설계하였습니다.

![플로차트 - 프레임 12](https://github.com/user-attachments/assets/45d02229-9006-40d5-b12a-4b1cb51c488e)

---

## 🗃️ Dataset

본 프로젝트의 핵심은 **유리 이미지로부터 손상 수준을 AI가 판단**하는 것입니다.

초기 제공된 데이터셋 구조는 다음과 같았습니다:

- Data  
  - Level of Damage  
    - Replace  
    - Repair



🔧 Repair Image Sample:  
![repair102](https://github.com/user-attachments/assets/0613852a-b1bc-46eb-8aca-7dd525d8393c)

🔨 Replace Image Sample:  
![replace78](https://github.com/user-attachments/assets/726d131a-e476-4fcb-ad99-8a298ad2c01c)

⚠️ **문제점**: 일부 클래스에는 **마킹 표시(스티커 등)**가 존재하여, 모델이 **표시 자체에 과적합(overfitting)** 될 위험이 있었습니다.  

➡️ 이에 따라, **YOLO 기반 객체 탐지 모델**을 활용하여  
**손상 부위만 추출(cropping)**한 새로운 데이터셋을 재구축하였습니다.

🔍 After YOLO Preprocessing:  
![replace78_crop0](https://github.com/user-attachments/assets/7fb61408-5b99-4bf0-8cc5-9121f81ab542)

---

## 🧪 Approach

전체 연구 흐름은 아래와 같습니다:

![1](https://github.com/user-attachments/assets/b32fa912-28a1-479d-a39b-37a364e7a9db)

### 🔨 단계별 구성

1. **YOLO 객체 탐지 모델 학습**
   - Roboflow를 통해 마킹 내 실제 손상 부위에 대해 **labeling**
   - Rotation, Flip 등 **Data Augmentation** 적용
   - YOLO v2, v4, v5, v8 모델을 각각 학습 후 성능 비교
   - 각 모델의 결과로부터 손상 부위만 **crop**하여 **새로운 데이터셋 구축**

2. **CNN 분류 모델 학습**
   - Cropping된 이미지로부터 손상 수준을 분류
   - **MobileNet v1, v2, v3-Large, VGG16, ResNet152** 모델 사용
   - 모델별 정확도 비교 및 성능 분석

---

# 📊 Result

## ✅ 신뢰도 향상 평가

본 연구의 가정은 다음과 같습니다:  
> "**모델이 마킹 표시 자체에 과적합**될 수 있어, 데이터의 **신뢰도가 낮다**."

따라서, YOLO 모델을 적용하여 신뢰도를 **얼마나 향상시킬 수 있는지** 측정합니다.

📐 **신뢰도 계산식**:  
`1 - (마킹이 표시된 이미지의 비율)`

📈 **YOLO 탐지 성능**:  
![2](https://github.com/user-attachments/assets/04fa91c9-c857-41d9-9430-8aba9bc2683b)

📊 **신뢰도 비교**:  
![Original_vs_YOLO_v2_v4_Reliability](https://github.com/user-attachments/assets/c917db04-8643-450b-ae08-6b3d486b9624)  
![Original_vs_YOLO_v5_v8_Reliability](https://github.com/user-attachments/assets/ce632c25-85d3-42ad-b825-426d6397fa5c)

> ✅ **최대 23.8%까지 신뢰도 향상**을 확인함

---

## 📈 분류(Classification) 성능 비교

🧪 Level of Damage 분류 결과:  
![Original_vs_YOLO_v2_v4_Classification(repair_vs_replace)](https://github.com/user-attachments/assets/11f97b5a-c8fc-470d-aa23-52604f479ae7)  
![Original_vs_YOLO_v5_v8_Classification (repair, replace)](https://github.com/user-attachments/assets/05a683c6-d7aa-46cf-986d-1e385795c1cd)

> 📉 **분류 성능은 1~3% 하락**  
> 🔁 이는 YOLO가 **데이터 복잡도를 증가시켰기 때문**으로 해석됨  
> ➕ 하지만 **성능 하락이 미미**하여 방법론적으로 **충분히 유의미**

---

## 🔍 복잡도 증가에 대한 검증 실험

YOLO가 실제로 **데이터의 복잡도를 높였는지**를 확인하기 위해  
손상 종류(Damage Type) 기반 분류 실험을 추가 수행했습니다.

📊 Damage Type Classification 결과:  
![Original_vs_YOLO_v2_v4_Classification(damagetype)](https://github.com/user-attachments/assets/7178d67f-ff2e-4e86-b434-df758d0d9c0a)  
![Original_vs_YOLO_v5_v8_Classification (damagetype)](https://github.com/user-attachments/assets/c02d06ee-753f-4035-90d9-da6dba5e98e3)

> 📉 성능이 **명확히 하락**함을 확인  
> ✔️ 이는 **YOLO로 인해 실제 데이터 복잡도가 상승했음**을 실증적으로 입증

---

# ✅ Summary

- ✅ **YOLO 기반 객체 탐지 모델**을 활용해 **데이터 내 과적합 요소(마킹)**를 제거
- ✅ **신뢰도 최대 23.8% 향상**, 분류 성능은 **미미한 하락(1~3%)**
- ✅ **데이터 복잡도 상승 효과 확인**, **모델 일반화 가능성 확보**
