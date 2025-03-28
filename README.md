# 📊 **암호화폐 대시보드 구축 프로젝트**
## 1️⃣ **데이터 수집 및 저장**

* 소스: CoinGecko API에서 최상위 100개 코인의 실시간 가격, 시가총액, 거래량 데이터 수집
* 데이터 저장: PostgreSQL 데이터베이스 내 coin_statistics 테이블에 저장
* 기간: 25/03/04~25/03/10
* 수집 주기: 1시간 단위로 업데이트
crypto_datacollection.ipynb 참조

## 2️⃣ 프로젝트 개요

* 목적: 암호화폐 거래소 또는 트레이딩 플랫폼이 데이터를 활용하여 사용자 경험을 개선하고, 시장 동향을 분석할 수 있도록 대시보드를 구축함. 또한 거래소의 주요 수익원인 거래량과 수수료를 기반으로, 시장 변동성과 장기 트렌드를 반영하여 거래소 수익을 예측할 수 있는 모델을 개발 
* 주요 기능:
  - 7일간 최고/최저 가격 변동률 분석
  - 시가총액 & 하락/상승 코인 비율 분석으로 전체 시장 흐름 파악
  - 시간대별 거래량 분석
  - 거래소 수익 예측 모델을 활용한 트레이딩 활성화 & 수익성 관계 분석  
* 활용 도구:
  - PostgreSQL을 사용하여 데이터 저장 및 쿼리
  - Pandas를 이용한 데이터 처리
  - Matplotlib 및 Seaborn을 활용한 데이터 시각화

## 3️⃣ 구축한 대시보드
### 📈 1. **[시장동향분석] 상승 하락 비율 분석 (24시간)**
* 설명: 24시간의 상승&하락 코인 비율 분석
* 시각화: 파이차트를 이용해 상승&하락 비율 표현 
* 활용방안
  - 급격한 가격 변동 대응: 상승/하락 비율이 크다면 리스크 경고 시스템을 통해 투자자들에게 변동성을 안내  
  - 단기 트레이딩 유도: 상승하는 코인이 많을 경우, 단기 트레이딩 수익 극대화를 위한 레버리지 상품이나 거래량 기반 리워드를 제공  
<img width="478" alt="image" src="https://github.com/user-attachments/assets/81a03421-35a6-4298-bd5a-f0ef630bf38d" />

### 📈 2. **[시장동향분석] 7일간의 전체 시가총액 흐름 분석**
* 설명: 전체 암호화폐 시장의 시가총액 변화를 7일간 분석하여 시장의 흐름을 파악, 시가총액은 시장의 전반적인 강세 또는 약세를 나타내는 중요한 지표로 활용되며, 투자자들의 시장 심리와 거래소의 수익성에도 큰 영향을 미침
* 시각화: 7일간의 시가총액 변화를 라인 차트로 시각화하여 상승 및 하락 추세를 직관적으로 확인할 수 있도록 구성
* 활용방안
  - 거래소 수익성 예측: 시가총액이 늘어날수록 거래 활성화로 거래소 수익이 증가할 가능성이 높으므로, 수익 예측과 사업 계획의 근거로 활용
  - 시장 위기관리 대응: 급격한 하락 추세 발생 시 투자자 보호를 위한 거래 제한이나 공지사항 배포 등 긴급 위기대응 조치를 조기에 준비
<img width="616" alt="image" src="https://github.com/user-attachments/assets/73e0a003-9a15-453e-abaf-1f7bf24012ec" />
  
### 📈 3. **[코인별분석] 7일간 최고/최저 가격 변동률 분석**
* 설명: 최근 7일 동안 특정 코인의 최고 가격과 최저 가격을 비교하여 변동률을 계산함. 변동성 높은 코인을 모니터링하고 유동성 관리 전략 조정 가능
* 시각화: 가로 바 차트를 이용하여 변동률이 높은 코인 표기
* 활용 방안:
  - 고변동성 코인 선정: 변동성이 높은 코인 프로모션을 통해 거래 활성화 유도
  - 리스크 관리: 특정 코인의 급격한 변동을 사전에 감지하여 시장 조작 가능성을 탐지하고 대응
  - 앱 푸시 활용: 변동성이 높은 코인을 감지하여 사용자에게 트레이딩 기회 제공  
  💬 예시 앱 푸시 알림: 📊 지난 7일간 가장 변동성이 높았던 코인은 $TRUMP! 가격 변동률 🚀 53%! 실시간 차트에서 확인하세요. 
* 적용된 데이터 필터링: 최저 가격의 50% 수치 (중앙값)을 기준으로 그 이하의 코인들은 삭제  

  ☑️ **중앙값을 사용한 이유**

    - 중앙값의 정의  
      중앙값은 데이터의 정중앙에 위치하는 값으로, 이상치나 극단치의 영향을 덜 받는 강건한(Robust) 통계적 척도

    - 코인시장 가격의 왜도 분석 (프로젝트 사용하고 있는 7일 데이터 기준)  
    코인 시장의 가격 데이터는 오른쪽으로 치우친(Positive Skewed) 분포를 보이므로, 평균보다는 중앙값을 사용하는 것이 적절함
    중앙값을 사용하면 극단적인 고가 코인들의 영향(예: 비트코인, 이더리움 등)을 줄이고, 전체적인 시장 가격 흐름을 더 안정적으로 나타낼 수 있음
    - 대표성 강화의 원리  
    시장의 주요 흐름이나 투자자의 관심이 집중되는 자산들은 일반적으로 시장 가격의 중간 이상 그룹에서 형성, 중앙값 이상의 데이터를 선택하면 실제 시장 흐름과 투자자의 행동을 더 정확히 반영할 가능성이 높아짐

<img width="616" alt="image" src="https://github.com/user-attachments/assets/c86f2425-fc11-4829-a011-2da4f05a56e2" />  

      

<img width="616" alt="image" src="https://github.com/user-attachments/assets/eefe66d3-6455-435b-b687-745077769d2e" />

### ⏰ 4. **시간대별 거래량 분석**
* 지난 7일동안의 시간/요일 별 거래량을 분석하여, 거래가 가장 활발한 시간대를 파악함
* 시각화: Heatmap 활용하여 요일 & 시간대별 거래량을 색상 차이로 표현함.
* 활용방안
  - 서버 부하 관리: 특정 시간대(예: 월요일 오후 9시~11시) 거래량이 급증하는 경우, 서버 자원을 미리 조정
  - 거래 수수료 최적화: 거래량이 적은 시간대에는 수수료 할인을 제공하여 유동성 증가 유도
  - 이벤트 타겟팅: 거래량이 가장 높은 요일과 시간대를 기반으로 거래 이벤트 또는 신규 토큰 상장을 기획
 <img width="615" alt="image" src="https://github.com/user-attachments/assets/6d7de178-a17e-456d-851c-54a6a1785921" />

* 하단 Redash 시각화 유관 코드 구현 해놓았지만 개인환경에서 Redash 실행 어려움으로 파이썬으로 시각화 진행
crypto_dashboard.ipynb 참조


# 📊 **암호화폐 대시보드 데이터 마트 구조**

## 🚀 **데이터 마트 구축 목표**
* 효율적인 분석을 위한 데이터 집계: 거래량, 시가총액, 가격 변동률 등을 집계하여 대시보드에서 빠르게 조회하고 시각화할 수 있도록 준비합니다.
* 이상치 필터링: IQR 방식으로 이상치를 제외하고, 과도한 변동을 제거하여 더 정확한 분석을 제공합니다.
* 시각화 및 대시보드: 집계된 데이터를 Redash나 다른 시각화 도구에서 사용할 수 있도록 저장하여 마케팅팀 등 실무자들이 필요한 정보를 실시간으로 확인할 수 있도록 합니다.

 ## 1️⃣ **데이터 마트 테이블 관계도**
|📑 테이블|📋 설명|
|------|---|
|coin_statistics|실시간 거래 기록 (가격, 시가총액, 거래량 등)|
|market trend|전체 시장 트렌드 분석 (상승/하락 비율, 시가총액)|
|market-cap trend|7일간의 전체 시가총액 흐름 분석|
|high_volatility_coins|7일간 변동성이 높은 코인 분석 (메이저 코인 필터링 포함)|
|hourly_volume|시간대별 거래량 분석|

## 2️⃣ **원본 테이블 (Staging Table)**

### 🏦 **`coin_statistics` 테이블**
- **목표**: 실시간 거래 데이터를 수집하여 기본 데이터의 원천으로 사용.
- **주요 컬럼**:
  - `coin_id` (VARCHAR): 코인의 고유 ID
  - `timestamp` (TIMESTAMP): 거래 시간
  - `price` (DOUBLE PRECISION): 코인 가격
  - `market_cap` (DOUBLE PRECISION): 시가총액
  - `volume` (DOUBLE PRECISION): 거래량
  - `circulating_supply` (DOUBLE PRECISION): 유통량
  - `total_supply` (DOUBLE PRECISION): 전체 공급량

```sql
CREATE TABLE coin_statistics (
    id SERIAL PRIMARY KEY,
    coin_id VARCHAR(50) NOT NULL,       -- 코인 ID
    timestamp TIMESTAMP NOT NULL,       -- 거래 시간
    price DOUBLE PRECISION NOT NULL,    -- 코인 가격
    market_cap DOUBLE PRECISION NOT NULL, -- 시가총액
    volume DOUBLE PRECISION NOT NULL,   -- 거래량
    circulating_supply DOUBLE PRECISION NOT NULL, -- 유통량
    total_supply DOUBLE PRECISION NOT NULL -- 전체 공급량
);
```
---

## 3️⃣ **집계된 데이터 테이블 (Aggregated Tables)**

### 💹 market_trend (📊 시장 트렌드)
- **목표**: 전24시간 동안 상승한 코인과 하락한 코인의 비율 분석
- **주요 컬럼**:
  - `timestamp` (TIMESTAMP): 거래 시간
  - `market_up_percentage` (DOUBLE PRECISION): 상승 코인의 비율
  - `market_down_percentage` (DOUBLE PRECISION): 하락 코인의 비율

```sql
-- market_trend 테이블 생성
CREATE TABLE market_trend (
    timestamp TIMESTAMP NOT NULL,       -- 거래 시간
    market_up_percentage DOUBLE PRECISION NOT NULL, -- 상승 코인의 비율
    market_down_percentage DOUBLE PRECISION NOT NULL -- 하락 코인의 비율
);

-- 최근 24시간 동안 상승 및 하락 코인의 비율 계산
INSERT INTO market_trend
SELECT
    NOW() AS timestamp,
    COUNT(CASE WHEN price > prev_price THEN 1 END) * 100.0 / COUNT(*) AS market_up_percentage,
    COUNT(CASE WHEN price < prev_price THEN 1 END) * 100.0 / COUNT(*) AS market_down_percentage
FROM (
    SELECT
        coin_id,
        timestamp,
        price,
        LAG(price) OVER (PARTITION BY coin_id ORDER BY timestamp) AS prev_price
    FROM coin_statistics
    WHERE timestamp >= NOW() - INTERVAL '24 hours'
) price_changes;
```
---

### 📈 **💰 market_cap_trend (📈 전체 시가총액 흐름) **
- **목표**: 최근 7일 동안 전체 암호화폐 시장의 시가총액 변동 분석
- **주요 컬럼**:
  - `timestamp` (timestamp): 거래 시간
  - `total_market_cap` (DOUBLE PRECISION): 전체 시장 시가총액

```sql    
-- market_cap_trend 테이블 생성
CREATE TABLE market_cap_trend (
    timestamp TIMESTAMP NOT NULL,       -- 거래 시간
    total_market_cap DOUBLE PRECISION NOT NULL -- 전체 시장 시가총액
);

-- 최근 7일간 시가총액 변동 데이터 삽입
INSERT INTO market_cap_trend
SELECT
    timestamp,
    SUM(price * volume) AS total_market_cap
FROM coin_statistics
WHERE timestamp >= NOW() - INTERVAL '7 days'
GROUP BY timestamp
ORDER BY timestamp;
```
---

### 📊 **high_volatility_coins (📈 변동성 높은 코인)**
- **목표**: 목표: 최근 7일간 최대/최소 가격 변화를 기반으로 변동성이 높은 코인 필터링
- **주요 컬럼**:
    - `coin_id` (VARCHAR): 코인 ID
    - `max_price_7d` (DOUBLE PRECISION): 최근 7일간 최고 가격
    - `min_price_7d` (DOUBLE PRECISION): 최근 7일간 최저 가격
    - `price_change_7d` (DOUBLE PRECISION): 가격 변동률 (%)

```sql 
-- high_volatility_coins 테이블 생성
CREATE TABLE high_volatility_coins (
    coin_id VARCHAR NOT NULL,            -- 코인 ID
    max_price_7d DOUBLE PRECISION NOT NULL,  -- 최근 7일간 최고 가격
    min_price_7d DOUBLE PRECISION NOT NULL,  -- 최근 7일간 최저 가격
    price_change_7d DOUBLE PRECISION NOT NULL -- 가격 변동률 (%)
);

-- 최근 7일간 변동성이 높은 코인 데이터 삽입
INSERT INTO high_volatility_coins
WITH weekly_extreme_prices AS (
    SELECT
        coin_id,
        MAX(price) AS max_price_7d,
        MIN(price) AS min_price_7d
    FROM coin_statistics
    WHERE timestamp >= NOW() - INTERVAL '7 days'
    GROUP BY coin_id
)
SELECT
    coin_id,
    max_price_7d,
    min_price_7d,
    (max_price_7d - min_price_7d) / min_price_7d * 100 AS price_change_7d
FROM weekly_extreme_prices
ORDER BY price_change_7d DESC;
```

---
⏰ **hourly_trading_volume (📊 시간대별 거래량)**
- **목표**: 목표: 최근 7일간 최대/최소 가격 변화를 기반으로 변동성이 높은 코인 필터링
- **주요 컬럼**:

    - `day_of_week` (INTEGER): 요일 (0=일요일, 6=토요일)
    - `hour` (INTEGER): 시간 (0~23)
    - `total_volume` (DOUBLE PRECISION): 총 거래량

```sql
-- hourly_trading_volume 테이블 생성
CREATE TABLE hourly_trading_volume (
    day_of_week INTEGER NOT NULL,       -- 요일 (0=일요일, 6=토요일)
    hour INTEGER NOT NULL,              -- 시간 (0~23)
    total_volume DOUBLE PRECISION NOT NULL -- 총 거래량
);

-- 최근 7일간 시간대별 거래량 데이터 삽입
INSERT INTO hourly_trading_volume
SELECT
    EXTRACT(DOW FROM timestamp) AS day_of_week,
    EXTRACT(HOUR FROM timestamp) AS hour,
    SUM(volume) AS total_volume
FROM coin_statistics
WHERE timestamp >= NOW() - INTERVAL '7 days'
GROUP BY day_of_week, hour
ORDER BY day_of_week, hour;
```
---
# 📊 **Redash 대시보드 구성**
## 1️⃣ **전체 시장 트렌드 분석**
목표:
상승/하락 비율과 전체 시가총액을 추적하여 시장 트렌드를 파악합니다.
파이 차트와 라인 차트를 사용하여 시장의 전반적인 변화를 시각화합니다.

### 1-1. **파이 차트: 상승/하락 비율**
* 목표: 상승하는 코인과 하락하는 코인의 비율을 계산하여 파이 차트로 표시합니다.
* SQL 쿼리:

```sql
SELECT 
    NOW() AS timestamp,
    COUNT(CASE WHEN price > prev_price THEN 1 END) * 100.0 / COUNT(*) AS market_up_percentage,
    COUNT(CASE WHEN price < prev_price THEN 1 END) * 100.0 / COUNT(*) AS market_down_percentage
FROM (
    SELECT 
        coin_id,
        timestamp,
        price,
        LAG(price) OVER (PARTITION BY coin_id ORDER BY timestamp) AS prev_price
    FROM coin_statistics
    WHERE timestamp >= NOW() - INTERVAL '24 hours'
) price_changes;
```

* Redash 설정:
시각화: 파이 차트를 선택하고 market_up_percentage와 market_down_percentage를 값으로 사용하여 상승과 하락 비율을 시각화합니다.

### 1-2. **라인 차트: 전체 시가총액 변화**
목표: 전체 시가총액의 변화를 추적하여 시장의 전반적인 방향을 라인 차트로 표시합니다.
SQL 쿼리:

```sql
SELECT 
    timestamp,
    SUM(price * volume) AS total_market_cap
FROM coin_statistics
WHERE timestamp >= NOW() - INTERVAL '7 days'
GROUP BY timestamp
ORDER BY timestamp;
```

* Redash 설정:
라인 차트를 선택하고 timestamp를 x축, total_market_cap을 y축으로 설정하여 시가총액 변화를 시각화합니다.

## 2️⃣ **7일 가격 변동률 상위 5개 코인**

* 목표:
7일 동안의 가격 변동률을 계산하고 상위 5개 코인을 추출합니다.
IQR 방식으로 거래량의 이상치를 제외하고 상위 5개 코인을 보여줍니다.
* SQL 쿼리:

```sql
WITH volume_stats AS (
    SELECT
        coin_id,
        PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY volume) AS q1,
        PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY volume) AS q3
    FROM coin_statistics
    WHERE timestamp BETWEEN NOW() - INTERVAL '7 days' AND NOW()
    GROUP BY coin_id
),
filtered_data AS (
    SELECT
        cs.coin_id,
        cs.price,
        cs.timestamp,
        cs.market_cap,
        cs.volume,
        LAG(cs.price) OVER (PARTITION BY cs.coin_id ORDER BY cs.timestamp) AS prev_price
    FROM coin_statistics cs
    WHERE cs.timestamp BETWEEN NOW() - INTERVAL '7 days' AND NOW()
)
SELECT 
    fd.coin_id,
    (fd.price - fd.prev_price) / fd.prev_price * 100 AS price_change_7d,  -- 7일 변동률
    fd.market_cap,
    fd.volume
FROM filtered_data fd
JOIN volume_stats vs
    ON fd.coin_id = vs.coin_id
WHERE fd.volume BETWEEN vs.q1 - 1.5 * (vs.q3 - vs.q1) AND vs.q3 + 1.5 * (vs.q3 - vs.q1)
ORDER BY price_change_7d DESC
LIMIT 5;
```
* Redash 설정:
시각화: 테이블 차트를 선택하고 coin_id, price_change_7d, market_cap, volume을 표시하여 상위 5개 코인의 정보를 보여줍니다.

3️⃣ **시간대별 거래량 변화**

* 목표:
시간대별 거래량 변화를 시각화하여 가장 거래량이 많은 시간대를 파악합니다.

* SQL 쿼리:

```sql
SELECT 
    EXTRACT(DOW FROM timestamp) AS day_of_week,
    EXTRACT(HOUR FROM timestamp) AS hour,
    SUM(volume) AS total_volume
FROM coin_statistics
WHERE timestamp >= NOW() - INTERVAL '7 days'
GROUP BY day_of_week, hour
ORDER BY day_of_week, hour;
```

* Redash 설정:
시각화: 라인 차트를 선택하고 hour를 x축, total_volume을 y축으로 설정하여 시간대별 거래량 변화를 시각화합니다.
