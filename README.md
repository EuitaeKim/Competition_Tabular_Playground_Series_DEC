# Tabular Playground Series(TPS)-December

### 1. 개요
Kaggle에서는 매월 1일부터 한 달 간의 Competition을 개최하고 있습니다. 매월 진행되는 만큼 Prediction 및 Classification의 다양한 문제 유형과 RMSE, ROC curve, Accuracy 등의 평가 지표를 활용하여 Score를 평가합니다. 때문에 초,중급자를 대상으로 Skill Up을 위한 최적의 Competition으로 평가받고 있습니다.

TPS에서 제공하는 데이터는 Tabular 형태로 제공되며, 대체로 비 식별화되어 있는 것이 특징입니다. 그러나 비식별화 여부와 관계 없이 EDA, Visualization부터 Feature Engineering, Statistical Analysis, ML/DL Engineering까지의 종합적인 실험을 통해 Score를 높힐 수 있는 특징을 가지고 있습니다. 저는 TPS 9월과 12월 대회에 솔로로 참여하여 각각 466/1942, 97/1188의 순위를 기록하였습니다. 그 중 다양한 실험을 진행했던 12월 대회의 내용을 정리하였습니다.

### 2. 대회 요약
1. 주제: Forest Cover Type Prediction
2. 목표: 콜로라도 북부의 루즈벨트 국유림에 위치한 4개의 야생 지역 관련 지도 제작에 사용된 다양한 변수를 활용하여 30m x 30m 크기 별로 구분 된 숲의 Cover Type을 분류
3. 문제 유형: Multi Class Classification
4. 데이터: 이전 대회에서 활용된 데이터를 CTGAN에 학습시켜 재생산한 데이터 (비 식별화처리 되지 않음)
5. 데이터 크기: Train(4000000, 56), Test(1000000, 55)

### 3. 데이터
|Column Name|Description-En|Description-Kor|Column Type|
|:---|:---|:---|:---|
|Elevation|Elevation in meters|고도(미터)|continuous|
|Aspect|Aspect in degrees azimuth|방위각의 종횡비|continuous|
|Slope|Slope in degrees|기울기(도)|continuous|
|Horizontal_Distance_To_Hydrology|Horz Dist to nearest surface water features|가장 가까운 지표수 피쳐까지의 수평 거리|continuous|
|Vertical_Distance_To_Hydrology|Vert Dist to nearest surface water features|가장 가까운 지표수 피쳐까지의 수직 거리|continuous|
|Horizontal_Distance_To_Roadways|Horz Dist to nearest roadway|가장 가까운 도로까지의 수평 거리|continuous|
|Hillshade_9am|Hillshade index at 9am, summer solstice|오전 9시 Hillshade 지수|(0 to 255 index),continuous|
|Hillshade_Noon|Hillshade index at noon, summer solstice|정오의 Hillshade 지수|(0 to 255 index),continuous|
|Hillshade_3pm|Hillshade index at 3pm, summer solstice|오후 3시 Hillshade 지수|(0 to 255 index),continuous|
|Horizontal_Distance_To_Fire_Points|Horz Dist to nearest wildfire ignition points|가장 가까운 산불 발화점까지의 수평 거리|continuous|
|Wilderness_Area, 1-4*|Wilderness area designation|황야 지역 지정|binary columns, 0 = absence or 1 = presence,continuous|
|Soil_Type, 1-40**|Soil Type designation|토양 유형|40 binary columns, 0 = absence or 1 = presence,continuous|
|Cover_Type***|Forest Cover Type designation|산림 유형 지정|7 types, integers 1 to 7,continuous|

## 1.대회 요약
### 1-1.개요
Kaggle에서는 2017년부터 매년 데이터 과학 및 머신러닝 분야의 포괄적인 관점 및 관련 정보를 제공하기 위해 업계 전반을 대상으로 설문조사를 진행해왔습니다. 올해의 설문조사는 2021년 9월 1일부터 10월 4일까지 진행되었고, 데이터 정제 후 **총 25,973건의 Survey Data를 수집**하였습니다. <br><br>

### 1-2.데이터 및 대회 안내
이러한 Kaggle Survey Data는 다양한 산업 분야에서 머신러닝과 관련해 어떤 일이 일어나고 있는지, 데이터와 함께 일하고 있는 사람은 어떤 사람인지, 새로운 데이터 과학자가 해당 분야에 진출할 수 있는 최선의 방법은 어떤 것인지 등의 정보를 포함하고 있습니다. 이번 대회는 데이터 과학 및 머신러닝에 대한 풍부한 이야기를 전하는 참가자에게 상금을 수여합니다. 즉, 내러티브 텍스트와 데이터 분석의 조합을 통해 이 설문조사에서 대표되는 데이터 과학에 대한 데이터 스토리를 전달하는 것이 이번 대회의 목표입니다. (ex: 특정 그룹의 영향, 우선순위 등) <br><br>

### 1-3.평가요소
- a. 구성 - 주제는 잘 정의되고 조사되어야 하며, 데이터와 시각화를 통해 잘 뒷받침되어야 한다. <br>
**Q. Is there a clear narrative thread to the story that’s articulated and supported by data?**
- b. 독창성 - 훌륭한 항목은 유익하고 생각을 자극하며 동시에 신선하다. <br>
**Q. Does the reader learn something new through this submission? Or is the reader challenged to think about something in a new way?**
- c. 문서화 - 고품질 분석은 각 단계에서 간결하고 명확하기 때문에 근거를 따르기 쉽고, 프로세스도 쉽게 재현할 수 있음을 뜻한다. <br>
**Q. Are your code, and notebook, and additional data sources well documented so a reader can understand what you did?** <br><br>

### 1-4. 기타
참가자는 Kaggle Survey Data 설문조사 외에 모든 데이터 세트를 자유롭게 사용할 수 있지만, 추가 데이터 세트는 제출 마감일까지 Kaggle에서 공개적으로 사용할 수 있어야 합니다.
