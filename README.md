# 📊 **암호화폐 대시보드 구축 프로젝트**

# 📊 **암호화폐 대시보드 데이터 마트 구조**

## 🚀 **데이터 마트 구축 목표**
* 효율적인 분석을 위한 데이터 집계: 거래량, 시가총액, 가격 변동률 등을 주기적으로 집계하여 대시보드에서 빠르게 조회하고 시각화할 수 있도록 준비합니다.
* 이상치 필터링: IQR 방식으로 이상치를 제외하고, 과도한 변동을 제거하여 더 정확한 분석을 제공합니다.
* 시각화 및 대시보드: 집계된 데이터를 Redash나 다른 시각화 도구에서 사용할 수 있도록 저장하여 마케팅팀이 필요한 정보를 실시간으로 확인할 수 있도록 합니다.

 ## 1️⃣ **데이터 마트 테이블 관계도**
|📑 테이블|📋 설명|
|------|---|
|coin_statistics|실시간 거래 기록 (가격, 시가총액, 거래량 등)|
|trend_analysis|전체 시장 트렌드 분석 (상승/하락 비율, 시가총액)|
|top_price_change_7d|7일 가격 변동률 상위 5개 코인 분석 (IQR 필터링 포함)|
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

### 💹 **`trend_analysis` 테이블**
- **목표**: 전체 시장의 **상승/하락 비율**과 **시가총액 변화를 추적**하여 분석합니다.
- **주요 컬럼**:
  - `timestamp` (TIMESTAMP): 거래 시간
  - `total_market_cap` (DOUBLE PRECISION): 전체 시장 시가총액
  - `market_up_percentage` (DOUBLE PRECISION): 상승 코인의 비율
  - `market_down_percentage` (DOUBLE PRECISION): 하락 코인의 비율

```sql
-- trend_analysis 테이블 생성
CREATE TABLE trend_analysis (
    timestamp TIMESTAMP NOT NULL,       -- 거래 시간
    total_market_cap DOUBLE PRECISION NOT NULL, -- 전체 시장 시가총액
    market_up_percentage DOUBLE PRECISION NOT NULL, -- 상승 코인의 비율
    market_down_percentage DOUBLE PRECISION NOT NULL -- 하락 코인의 비율
);

-- 상위 1개 코인마다 시가총액과 상승/하락 비율을 계산합니다.
WITH price_changes AS (
    SELECT 
        coin_id,
        timestamp,
        price,
        volume,
        LAG(price) OVER (PARTITION BY coin_id ORDER BY timestamp) AS prev_price,
        CASE 
            WHEN price > LAG(price) OVER (PARTITION BY coin_id ORDER BY timestamp) THEN 1 
            ELSE 0 
        END AS market_up,
        CASE 
            WHEN price < LAG(price) OVER (PARTITION BY coin_id ORDER BY timestamp) THEN 1 
            ELSE 0 
        END AS market_down
    FROM coin_statistics
    WHERE timestamp >= '2025-03-07 00:00:00' 
      AND timestamp < '2025-03-08 00:00:00'
)
INSERT INTO trend_analysis (timestamp, total_market_cap, market_up_percentage, market_down_percentage)
SELECT 
    timestamp,
    SUM(price * volume) AS total_market_cap,
    (SUM(market_up) / COUNT(*)) * 100 AS market_up_percentage,  -- 상승 비율
    (SUM(market_down) / COUNT(*)) * 100 AS market_down_percentage -- 하락 비율
FROM price_changes
GROUP BY timestamp
ORDER BY timestamp;
```
---

### 📈 **`top_price_change_7d` 테이블**
- **목표**: 7일 동안의 **가격 변동률**을 계산하고 **상위 5개 코인**을 추적합니다. 이때, **IQR 방식**으로 거래량의 **이상치**를 제외합니다.
- **주요 컬럼**:
  - `coin_id` (VARCHAR): 코인 ID
  - `price_change_7d` (DOUBLE PRECISION): 7일 가격 변동률
  - `market_cap` (DOUBLE PRECISION): 시가총액
  - `volume` (DOUBLE PRECISION): 거래량
```sql    
CREATE TABLE top_price_change_7d (
    coin_id VARCHAR(50) NOT NULL,     -- 코인 ID
    price_change_7d DOUBLE PRECISION, -- 7일 가격 변동률
    market_cap DOUBLE PRECISION,      -- 시가총액
    volume DOUBLE PRECISION,          -- 거래량
    PRIMARY KEY (coin_id)
);

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
INSERT INTO top_price_change_7d (coin_id, price_change_7d, market_cap, volume)
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
---

### 📅 **`hourly_volume` 테이블**
- **목표**: **시간대별 거래량**을 추적하여 마케팅 캠페인 시간을 최적화합니다.
- **주요 컬럼**:
  - `hour` (INT): 시간대 (0~23)
  - `total_volume` (DOUBLE PRECISION): 해당 시간대의 총 거래량

```sql   sql   
CREATE TABLE hourly_volume (
    hour INT NOT NULL,               -- 시간대 (0~23)
    total_volume DOUBLE PRECISION,   -- 해당 시간대의 총 거래량
    PRIMARY KEY (hour)
);

-- 시간대별 거래량을 계산합니다.
INSERT INTO hourly_volume (hour, total_volume)
SELECT 
    EXTRACT(HOUR FROM timestamp) AS hour,
    SUM(volume) AS total_volume
FROM coin_statistics
WHERE timestamp BETWEEN NOW() - INTERVAL '7 days' AND NOW()
GROUP BY hour
ORDER BY hour;
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
    timestamp,
    (SUM(CASE WHEN price > LAG(price) OVER (PARTITION BY coin_id ORDER BY timestamp) THEN 1 ELSE 0 END) / COUNT(*)) * 100 AS market_up_percentage,
    (SUM(CASE WHEN price < LAG(price) OVER (PARTITION BY coin_id ORDER BY timestamp) THEN 1 ELSE 0 END) / COUNT(*)) * 100 AS market_down_percentage
FROM coin_statistics
GROUP BY timestamp
ORDER BY timestamp;
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
GROUP BY timestamp
ORDER BY timestamp;
```

* Redash 설정:
시각화: 라인 차트를 선택하고 timestamp를 x축, total_market_cap을 y축으로 설정하여 시가총액 변화를 시각화합니다.

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
    EXTRACT(HOUR FROM timestamp) AS hour,
    SUM(volume) AS total_volume
FROM coin_statistics
WHERE timestamp BETWEEN NOW() - INTERVAL '7 days' AND NOW()
GROUP BY hour
ORDER BY hour;
```

* Redash 설정:
시각화: 라인 차트를 선택하고 hour를 x축, total_volume을 y축으로 설정하여 시간대별 거래량 변화를 시각화합니다.
