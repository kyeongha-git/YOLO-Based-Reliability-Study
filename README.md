# Introduction

본 리포는 2024년 한국정보과학회에 투고한 『운전자 안전을 위한 차량 유리 복원 교체 판단 모델 구축에서 과적합 가능성을 가진 데이터로부터 YOLO 기반 신뢰성 확보 연구』 논문에 대한 내용과 이후 공개할 『객체 탐지 기반 전처리를 활용한 차량 유리 손상 분류 신뢰도 향상 방안』 논문에 대한 코드 및 결과를 담고 있습니다.

# Description

## Situation

 국토교통부에 따르면, 2023년 기준 국내 등록 자동차 대수는 2014년 대비 약 5백만 대가 증가했습니다. 이는 도로의 복잡성을 높이고 사고 발생률을 높입니다. 그만큼, 사고 이후 대처 방안도 중요합니다.
카센터에 들러 자동차를 수리할 때 과도한 비용이 청구되면 사람들은 한 번씩 생각합니다. "너무 많이 나온 것 같은데 .. 내가 몰라서 뭐라고 할 수가 없네 ..." 
이런 상황을 대처하기 위해서 수리비 견적을 미리 AI를 통해 받는다면, 이렇게 과도하게 청구된 비용에 대한 사람들의 억울함은 줄어들고 소위 말하는 "바가지" 장사도 줄어들 것입니다.
이에, 저는 (주)에스에이시스템에서 제공받은 데이터로 자동차 유리에 대한 손상 정도를 AI를 통해 판별하는 아키텍처를 구성했습니다.

![플로차트 - 프레임 12](https://github.com/user-attachments/assets/45d02229-9006-40d5-b12a-4b1cb51c488e)

## Dataset

 본 프로젝트에 핵심은 AI 모델이 유리 이미지를 보고 손상 정도를 판별하는 것입니다. 그러나, 저희가 제공받은 데이터셋은 다음과 같았습니다.
Data
ㄴ Level of Damage
  ㄴ Replace
  ㄴ Repair

Repair Iamge sample:

![repair102](https://github.com/user-attachments/assets/0613852a-b1bc-46eb-8aca-7dd525d8393c)

Replace Image sample:
![replace78](https://github.com/user-attachments/assets/726d131a-e476-4fcb-ad99-8a298ad2c01c)

이처럼 특정 클래스에만 마킹 표시가 존재하였기 때문에, 모델이 마킹 표시에 과적합 될 우려가 존재했습니다. 따라서, 본 연구는 이런 우려를 제거하기 위하여 Object Detection 모델을 활용해 마킹 표시 내 존재하는 손상 부위만 추출하여 다시 데이터셋을 재구축하였습니다.

After YOLO Preprocessing:
![replace78_crop0](https://github.com/user-attachments/assets/7fb61408-5b99-4bf0-8cc5-9121f81ab542)

## Approach

 저의 연구의 로직은 아래와 같습니다.
![1](https://github.com/user-attachments/assets/b32fa912-28a1-479d-a39b-37a364e7a9db)

이를 위해선 두 가지 방향의 연구가 진행되어야 합니다.
1) YOLO 모델 학습
2) CNN 모델 학습

먼저, YOLO 모델을 학습하기 위해서 제공받은 데이터셋에 Roboflow labeling 프로그램을 활용해 마킹 내 부위에 labeling 해주었습니다. 이후, Data Augmentation (Rotation, Flip 등) 진행하여 학습 데이터셋을 구축합니다.

Detection Performance는 모델 별로 상이하기에 YOLO v2, v4, v5, v8을 각각 학습하여 성능을 비교하고 각 모델을 활용해 Cropping Dataset을 구축합니다.

구축된 데이터셋을 활용해 CNN 모델을 학습합니다. 다양한 모델의 성능을 파악하기 위하여, MobileNet v1, v2, v3 Large, VGG16, ResNet152를 사용하여 각 모델을 훈련하고 정확도를 평가합니다.

## Result

 우리의 가정은 모델이 마킹 표시에 과적합 되어 잘못 판단할 우려가 있다는 것입니다. 이를 우리는 **데이터의 신뢰도가 떨어진다**라고 표현합니다. 따라서, YOLO 모델을 통해 얼마나 이 신뢰도가 향상했는지를 확인합니다. 이는 1 - (마킹이 표시된 비율) 로 계산합니다.

먼저, YOLO 모델의 성능은 다음과 같습니다.

![2](https://github.com/user-attachments/assets/04fa91c9-c857-41d9-9430-8aba9bc2683b)

아래는 신뢰도 비교 결과입니다.

 ![Original_vs_YOLO_v2_v4_Reliability](https://github.com/user-attachments/assets/c917db04-8643-450b-ae08-6b3d486b9624)

![Original_vs_YOLO_v5_v8_Reliability](https://github.com/user-attachments/assets/ce632c25-85d3-42ad-b825-426d6397fa5c)

이후, Classification 성능을 측정합니다.
![Original_vs_YOLO_v2_v4_Classification(repair_vs_replace)](https://github.com/user-attachments/assets/11f97b5a-c8fc-470d-aa23-52604f479ae7)

![Original_vs_YOLO_v5_v8_Classification (repair, replace)](https://github.com/user-attachments/assets/05a683c6-d7aa-46cf-986d-1e385795c1cd)

 보이는 것처럼 YOLO 모델을 사용하여 **신뢰도가 최대 23.8% 상승**했음을 확인하였으며, **분류 성능은 1~3% 하락**하였음을 확인했습니다. 이는 YOLO 모델이 데이터의 복잡도를 향상시켰음에 기인한다고 보이며 신뢰도와 Trade-Off 관계를 보이지만, 하락 정도가 미미하여 방법론 측면에서 유의미합니다.

또한, YOLO 모델이 정말 데이터의 복잡도를 올리는가를 판별하기 위하여 손상 종류 데이터를 생성하고 이를 YOLO Cropping을 거쳐 다시 Classification을 진행했습니다.
![Original_vs_YOLO_v2_v4_Classification(damagetype)](https://github.com/user-attachments/assets/7178d67f-ff2e-4e86-b434-df758d0d9c0a)

![Original_vs_YOLO_v5_v8_Classification (damagetype)](https://github.com/user-attachments/assets/c02d06ee-753f-4035-90d9-da6dba5e98e3)

그 결과, 정확도가 크게 하락하는 것으로 보여 위 성능 하락이 데이터의 복잡도를 상승시키는 것에서 기인한 것이 맞음을 확인하였습니다.
