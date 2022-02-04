# Tabular Playground Series(TPS)-December

## 1. 개요
<img width="970" alt="1" src="https://user-images.githubusercontent.com/66727848/152116299-1ee3abd6-0dd6-4d92-90a6-809b023903ab.png">

Kaggle은 매월 1일부터 한 달 간 Competition을 개최합니다. 매월 진행되는 만큼 Prediction 및 Classification의 다양한 문제 유형과 RMSE, ROC curve, Accuracy 등의 평가 지표를 활용하여 Score를 평가합니다. 때문에 초, 중급자를 대상으로 Skill Up을 위한 최적의 Competition으로 평가받고 있습니다.

TPS에서 제공하는 데이터는 Tabular 형태로 제공되며, 대체로 비 식별화되어 있는 것이 특징입니다. 그러나 비식별화 여부와 관계 없이 EDA, Visualization부터 Feature Engineering, ML/DL Engineering까지의 종합적인 실험을 통해 Score를 높힐 수 있는 특징을 가지고 있습니다. 저는 TPS 9월과 12월 대회에 솔로로 참여하여 각각 466/1942, 97/1188의 순위를 기록하였습니다. 그 중 다양한 실험을 진행했던 12월 대회의 내용을 정리하였습니다.
</br></br>
## 2. 대회 요약
1. 주제: Forest Cover Type Prediction
2. 목표: 콜로라도 북부의 루즈벨트 국유림에 위치한 야생 지역 관련 지도 제작에 사용된 다양한 변수를 활용하여 30m x 30m 크기 별로 구분 된 숲의 Cover Type을 분류
3. 문제 유형: Multi Class Classification
4. 데이터: 이전 대회에서 활용된 데이터를 CTGAN에 학습시켜 재생산한 데이터로, 비 식별화처리 되지 않음
5. 데이터 크기: Train(4000000, 56), Test(1000000, 55)
</br></br>
## 3. 데이터
|Column Name|Description-En|Description-Kor|Column Type|
|:---|:---|:---|:---|
|Elevation|Elevation in meters|고도(미터)|continuous|
|Aspect|Aspect in degrees azimuth|방위각의 종횡비|continuous|
|Slope|Slope in degrees|기울기(도)|continuous|
|Horizontal_Distance_To_Hydrology|Horz Dist to nearest surface water features|가장 가까운 지표수 피쳐까지의 수평 거리|continuous|
|Vertical_Distance_To_Hydrology|Vert Dist to nearest surface water features|가장 가까운 지표수 피쳐까지의 수직 거리|continuous|
|Horizontal_Distance_To_Roadways|Horz Dist to nearest roadway|가장 가까운 도로까지의 수평 거리|continuous|
|Hillshade_9am|Hillshade index at 9am, summer solstice|오전 9시 Hillshade 지수|continuous(0 to 255, index)|
|Hillshade_Noon|Hillshade index at noon, summer solstice|정오의 Hillshade 지수|continuous(0 to 255, index)|
|Hillshade_3pm|Hillshade index at 3pm, summer solstice|오후 3시 Hillshade 지수|continuous(0 to 255, index)|
|Horizontal_Distance_To_Fire_Points|Horz Dist to nearest wildfire ignition points|가장 가까운 산불 발화점까지의 수평 거리|continuous|
|Wilderness_Area, 1-4*|Wilderness area designation|황야 지역 지정|binary(0=absence, 1=presence)|
|Soil_Type, 1-40**|Soil Type designation|토양 유형|binary(0=absence, 1=presence)|
|Cover_Type***|Forest Cover Type designation|산림 유형 지정|categorical(1 to 7, index)|

**Wilderness_Area:**
1-Rawah Wilderness Area, 2-Neota Wilderness Area, 3-Comanche Peak Wilderness Area, 4-Cache la Poudre Wilderness Area

**Soil_Type:**
1-Cathedral family - Rock outcrop complex, extremely stony, 2-Vanet - Ratake families complex, very stony, 3-Haploborolis - Rock outcrop complex, rubbly, 4-Ratake family - Rock outcrop complex, rubbly, 5-Vanet family - Rock outcrop complex complex, rubbly, 6-Vanet - Wetmore families - Rock outcrop complex, stony…etc

**Cover_Type:**
1-Spruce/Fir, 2-Lodgepole Pine, 3-Ponderosa Pine, 4-Cottonwood/Willow, 5-Aspen, 6-Douglas-fir, 7-Krummholz
</br></br>
## 4. Continuous Column 인사이트 종합
### 4-1. 공통
1. column별 train/test 데이터 분포가 거의 유사
2. Aspect을 제외한 모든 column에서 많은 이상치 발견
3. Elevation을 제외한 모든 column에서 target label 구분이 거의 안됨
<img width="970" alt="1" src="https://user-images.githubusercontent.com/66727848/152116055-2388f858-5467-429a-850b-01248d02ada6.png">


### 4-2. 개별 인사이트

|Column Name|Issue|problem solving direction|
|:---|:---|:---|
|Aspect, Slope|각도, 경사를 의미하는데 < 0 or > 360 값 존재|1. 정상 범위 안으로 수정 2. +- 360, +-90으로 값 보정 3. 수정하지 않음|
|Distance|거리를 의미하는데 < 0 값 존재|1. 정상 범위 안으로 수정 2. 수정하지 않음|
|Hillshade|(유추)grayscale형식의 색상 값인데 < 0 or > 255 값 존재|1. 정상 범위 안으로 수정 2. 수정하지 않음|

### 4-3. 개별 인사이트 세부

|Column Name|Case.1-train|Case.2-test|Case.3-train|Case.4-test|
|:---|:---|:---|:---|:---|
|Aspect|train<0: 43,730개|test<0: 24,948개|train>360: 68,323개|test>360: 17,946개|
|Slope|train<0: 7,822개|test<0: 2,027개|-|-|
|Horizontal_Distance_To_Hydrology|train<0: 2,959개|test<0: 4,123개|-|-|
|Vertical_Distance_To_Hydrology|train<0: 567,497개|test<0: 145,050개|-|-|
|Horizontal_Distance_To_Roadways|train<0: 30,726개|test<0: 10,160개|-|-|
|Horizontal_Distance_To_Fire_Points|train<0: 25,717개|test<0: 6,246개|-|-|
|Hillshade_9am|train<0: 6개|-|train>255: 32,128개|test>255: 9,243개|
|Hillshade_Noon|-|-|train>255: 38,861개|test>255: 10,997개|
|Hillshade_3pm|train<0: 14,211개|test<0: 4,485개|train>255: 4,277개|test>255: 1,094개|

</br>

## 5. Categorical Column 인사이트 종합
1. Cover_Type(target) column에서 데이터가 1개(5) 혹은 극소수인(4) label이 있음
2. Soil_Type7, Soil_Type15 column은 데이터 값 하나만을 갖음
3. Wilderness_Area2을 제외한 모든 column에서의 결괏값이 불균형함
<img width="970" alt="1" src="https://user-images.githubusercontent.com/66727848/152118126-b5694973-2033-4054-a9e4-4ed561f2bfd0.png">

</br>

## 6. Feature Engineering 종합
<img width="970" alt="1" src="https://user-images.githubusercontent.com/66727848/152330383-5163a995-91a4-416d-a3b4-a2d6503fce93.png">

</br>

## 7. Modeling 결과 종합 (Hyperparameter & Feature Importances)
### 7-1. Hyperparameter Importances
1. Hyperparameter Importances의 chart를 보면, 모델 성능에 큰 영향을 주는 Parameter를 ‘Learning_rate’, ‘Max_depth’임을 알 수 있다.
2. ‘Learning_rate’, ‘Max_depth’의 Value 별 Accuracy를 나타낸 Slice Plot을 보면, ‘Learning_rate’는 값이 커질수록, ‘Max_depth’은 값이 작을수록 좋은 성능을 내고 있다.
3. 위의 인사이트를 바탕으로 모델을 더 이상 깊게, 혹은 디테일하게 디벨롭할 필요가 없다 판단하였다. 즉, 모델 튜닝으로는 성능 향상의 한계가 있으며, 데이터를 더 핸들링 해야한다고 판단하였다.
4. 때문에 추가로 Feature Importances를 진행하였다.

<img width="970" alt="1" src="https://user-images.githubusercontent.com/66727848/152130535-30c9d97e-91fb-4ca0-afd7-205714f08ced.png">

### 7-2. Feature Importances
1. 위 차트는 feature가 사용된 전체 노드의 평균 Gain이며, 아래 chart는 feature가 사용된 전체 노드 Gain의 총합이다.
2. 두 chart를 보면 어느 한쪽의 결괏값이 굉장히 낮거나, 두 chart 모두에서 낮은 값을 갖는 경우가 있다. 이러한 feature는 제거하였다. 다만 원본 feature는 모델의 학습 시간이 매우 커지는 경우를 제외하고는 제거하지 않았다.
3. 추가로 confusion matrix를 활용하여 Cover_Type=2와 3, Cover_Type=1과 7을 구분하지 못하는 것이 낮은 성능의 원인임을 파악하였다. 이러한 문제를 해결하기 위해 여러 feature를 만들어보고 제거도 해보았지만, 성능 향상에는 한계가 있었다. 때문에 기존 XGboost에서 딥러닝 모델로 변경하여 다시 모델링을 진행하였다.

<img width="970" alt="1" src="https://user-images.githubusercontent.com/66727848/152137371-bca09f36-2f59-482e-b572-7e14acb047df.png">

*참고: Importance 측정 단위를 Gain으로 설정한 이유*
1. XGBoost는 다양한 방법의 importance 측정 단위를 제공한다. 
2. Weight는 각 feature가 노드 분기에 사용된 횟수를, Cover은 각 feature가 관여한 샘플의 수를, Gain은 각 feature로 분기되었을 때 얻는 성능 상의 이득을 결과로 산출한다.
3. 이때 사용하는 데이터가 범주형이 많을 경우 Weight는 적절하지 않으며, Label 별 데이터양이 불균형한 경우 Cover도 적절하지 않기 때문에 Gain으로 설정하였다.

</br>

## 8. 최종 모델 및 학습 결과
<img width="970" alt="1" src="https://user-images.githubusercontent.com/66727848/152508591-2f2b877c-ceff-437d-bbb4-42e04807341c.png">


***Test data Accuracy: Public Leaderboard: 0.95711, Private Leaderboard: 0.95658***

## 9. 회고
1. 
