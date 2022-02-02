# Tabular Playground Series(TPS)-December

## 1. 개요
Kaggle에서는 매월 1일부터 한 달 간 Competition을 개최합니다. 매월 진행되는 만큼 Prediction 및 Classification의 다양한 문제 유형과 RMSE, ROC curve, Accuracy 등의 평가 지표를 활용하여 Score를 평가합니다. 때문에 초, 중급자를 대상으로 Skill Up을 위한 최적의 Competition으로 평가받고 있습니다.

TPS에서 제공하는 데이터는 Tabular 형태로 제공되며, 대체로 비 식별화되어 있는 것이 특징입니다. 그러나 비식별화 여부와 관계 없이 EDA, Visualization부터 Feature Engineering, Statistical Analysis, ML/DL Engineering까지의 종합적인 실험을 통해 Score를 높힐 수 있는 특징을 가지고 있습니다. 저는 TPS 9월과 12월 대회에 솔로로 참여하여 각각 466/1942, 97/1188의 순위를 기록하였습니다. 그 중 다양한 실험을 진행했던 12월 대회의 내용을 정리하였습니다.
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
1-Cathedral family - Rock outcrop complex, extremely stony, 2-Vanet - Ratake families complex, very stony </br>
3-Haploborolis - Rock outcrop complex, rubbly, 4-Ratake family - Rock outcrop complex, rubbly </br>
5-Vanet family - Rock outcrop complex complex, rubbly, 6-Vanet - Wetmore families - Rock outcrop complex, stony…etc

**Cover_Type:**
1-Spruce/Fir, 2-Lodgepole Pine, 3-Ponderosa Pine, 4-Cottonwood/Willow, 5-Aspen, 6-Douglas-fir, 7-Krummholz
</br></br>
## 4. Continuous Column 인사이트 종합
1. train/test 데이터의 분포가 거의 유사
2. Aspect을 제외한 모든 column에서 많은 이상치 발견
3. Elevation을 제외한 모든 column에서 target label 구분이 거의 안됨

### 4-1. 이슈 세부

|Column Name|Issue|problem solving direction|
|:---|:---|:---|
|Aspect, Slope|각도, 경사를 의미하는데 < 0 or > 360 값 존재|1. 정상 범위 안으로 수정 2. +- 360, +-90으로 값 보정 3. 변경하지 않음|
|Distance|거리를 의미하는데 < 0 값 존재|1. 정상 범위 안으로 수정 2. 변경하지 않음|
|Hillshade|(유추)grayscale형식의 색상 값인데 < 0 or > 255 값 존재|1. 정상 범위 안으로 수정 2. 변경하지 않음|

### 4-2. 데이터 세부

|Column Name|Case.1|Case.2|Case.3|Case.4|
|:---|:---|:---|:---|:---|
|Aspect|train<0: 43,730개|test<0: 24,948개|train>360: 68,323개|test>360: 17,946개|
|Slope|train<0: 7,822개|test<0: 2,027개|||
|Horizontal_Distance_To_Hydrology|train<0: 2,959개|test<0: 4,123개|||
|Vertical_Distance_To_Hydrology|train<0: 567,497개|test<0: 145,050개|||
|Horizontal_Distance_To_Roadways|train<0: 30,726개|test<0: 10,160개|||
|Horizontal_Distance_To_Fire_Points|train<0: 25,717개|test<0: 6,246개|||
|Hillshade_9am|train<0: 6개||train>255: 32,128개|test>255: 9,243개|
|Hillshade_Noon|||train>255: 38,861개|test>255: 10,997개|
|Hillshade_3pm|train<0: 14,211개|test<0: 4,485개|train>255: 4,277개|test>255: 1,094개|

</br>

## 5. Categorical Column 인사이트 종합
1. Cover_Type(target) column에서 데이터가 1개(5) 혹은 극소수인(4) label이 있음
2. Soil_Type7, Soil_Type15 column은 데이터 값 하나만을 갖음
3. Wilderness_Area2을 제외한 모든 column에서의 결괏값이 불균형함
