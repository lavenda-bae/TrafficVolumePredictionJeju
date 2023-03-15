# DACON


## 🚗 제주도 도로 교통량 예측 AI 경진대회

### ****📅 프로젝트 진행기간****

2022.10.26 ~ 22.11.14

### ****📔 프로젝트 내용****

제주도의 교통 정보로부터 도로 교통량 회귀 예측

### ****💪 역할****

- 여민희 : EDA, 모델링(LightGBM,Model Staking)
- 유한솔 : EDA , 모델링(CatBoost, Model Staking)
- 이재엽 : 파생변수 추출(상대유동지수), 모델링(LightGBM)
- 전은성 : 모델링(LightGBM, GradientBoosting, Xgboost, LSTM with Attention, Model Staking)

### ****🗄️ 데이터셋****

- train : 4,701,217개
- test : 291,241개
- columns 
 
   <img src="https://user-images.githubusercontent.com/104626180/202997045-7a679a11-f40b-4213-8a0d-f0c8bc2e3416.png" width="700" height="420">
- 외부 데이터셋 : [제주시 일일 단위 구간별 평균 통행 속도 정보](http://www.jejuits.go.kr/open_api/open_apiView.do) (API key 신청 후 사용)

### ****⚙️ Preprocess****

- EDA
    - correlation matrix(전체)
        - 상관계수가 높은 변수들이 많지 않음(0.2 이상인 변수 4개 존재)
            
            ![상관계수](https://user-images.githubusercontent.com/104626180/202994809-7f7c1d7c-1741-410d-b978-b884e1bfd110.png)
            
    - correlation matrix(수치형)
    
    ![수치형](https://user-images.githubusercontent.com/104626180/202994928-a50fbffa-385f-4650-bcd6-6063fe2bff81.png)
    
    - PointBiSerial correlation (명목형)
        
        ![명목형1](https://user-images.githubusercontent.com/104626180/202995102-e979bf31-3d48-4f3e-b292-b6fb14c0a44f.png)
        
        ![명목형2](https://user-images.githubusercontent.com/104626180/202995144-2f83c146-2cdb-46d5-a8a5-439cd15596ef.png)
        
    - 추이그래프
        - base_date = 2022년 7월 기준 교통량 증가
            
            
            ![base_date1](https://user-images.githubusercontent.com/104626180/202995257-d46402bf-55a3-4b49-9852-68d58141648c.png)
            
            ![base_date2](https://user-images.githubusercontent.com/104626180/202995306-0aa691be-f2d1-4679-bceb-57d3a85ac244.png)

            
    - base_hour = 00시-05시,18시-24시 교통량 감소, 05시-18시 교통량 증가 (차이가 큼)
        
        ![base_hour](https://user-images.githubusercontent.com/104626180/202995378-790ab728-c3e3-4df3-b68f-279ce438d055.png)
        
    - day_of_week = 금요일 교통량 증가, 주말 교통량 감소 (큰차이 없음)
        
        ![base_week](https://user-images.githubusercontent.com/104626180/202995461-3d67bbed-d5e9-4076-9bdf-c8d37a518230.png)
        
- 파생변수
    - distance(km 기준)
        - haversine과 시작점, 도착점의 위도 경도를 이용해서 거리를 구해줌
    - 금요일 여부(isfriday 컬럼)
        - 금요일일 경우 True, 아니면 False
    - 요일별 가중치(day_weight 컬럼)
        - 화,수,목 : 1
        - 월,금 : 2
        - 토, 일 : 3
    - 상대유동지수(OOO_mean_speed 컬럼)
        - hour, day, road, lane, max_speed, road_rating, road_type 별로 평균속도를 구함
    - 7월 상대유동지수(OOO_mean_july_speed 컬럼)
        - 2022년 7월 기준 교통량 증가 → 7월의 상대유동지수 추가
        - lane, max_speed, road_rating, road_type 별로 평균속도를 구함
    - 19년도 월별 통행속도
 - polynomial features 사용

### 📝 ****Modeling****

- dataset
    - 7월 데이터(distance + 상대유동지수 추가)
        - 7월 데이터로만 학습했을 때 성능이 제일 잘나왔음
    - 7월 데이터(distance + 상대유동지수 + isfriday 추가)
    - 7월 16일 이후 데이터(distance + 유동지수 추가)
        - 7월 16일 이후 다른 추이를 보여서 나눠봄
        
        <img width="1074" alt="modeling" src="https://user-images.githubusercontent.com/104626180/202995601-4f2df3d9-1662-48d2-8cae-aac02f1f4a01.png">
        
    - 21년 9월 ~ 22년 6월 데이터 (distance + 상대유동지수 + 7월 상대유동지수 추가)
    - 7월 데이터(distance + 상대유동지수 + 7월 상대유동지수 추가)
    - 22년 1월 ~ 7월 데이터(distance + 상대유동지수 + 7월 상대유동지수 + 월별 평균속도 추가)
    - 22년 4월 ~ 7월 데이터(distance + 상대유동지수 + 7월 상대유동지수 + 월별 평균속도 추가)
- models
    - LightGBM, CatBoost, XGBoost, GradientBoosing
    - optuna를 사용해서 최적의 하이퍼파라미터를 찾음
- ensemble
    - 각각의 모델에서 나온 결과값을 mean, median을 이용해서 ensemble
    
    <img width="1020" alt="last" src="https://user-images.githubusercontent.com/104626180/202995684-d8a092ba-2ab4-40a0-a859-32438c97fc21.png">
    

### ****🏅 결과****

- private score :  3.09375
- public score : 3.09759
    - 9등 /  712등
