# 📊 **암호화폐 대시보드 구축 프로젝트**

# 📊 **암호화폐 대시보드 데이터 마트 구조**

## 1️⃣ **원본 테이블 (Staging Table)**

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

---

## 2️⃣ **집계된 데이터 테이블 (Aggregated Tables)**

### 💹 **`trend_analysis` 테이블**
- **목표**: 전체 시장의 **상승/하락 비율**과 **시가총액 변화를 추적**하여 분석합니다.
- **주요 컬럼**:
  - `timestamp` (TIMESTAMP): 거래 시간
  - `total_market_cap` (DOUBLE PRECISION): 전체 시장 시가총액
  - `market_up_percentage` (DOUBLE PRECISION): 상승 코인의 비율
  - `market_down_percentage` (DOUBLE PRECISION): 하락 코인의 비율

---

### 📈 **`top_price_change_7d` 테이블**
- **목표**: 7일 동안의 **가격 변동률**을 계산하고 **상위 5개 코인**을 추적합니다. 이때, **IQR 방식**으로 거래량의 **이상치**를 제외합니다.
- **주요 컬럼**:
  - `coin_id` (VARCHAR): 코인 ID
  - `price_change_7d` (DOUBLE PRECISION): 7일 가격 변동률
  - `market_cap` (DOUBLE PRECISION): 시가총액
  - `volume` (DOUBLE PRECISION): 거래량

---

### 📅 **`hourly_volume` 테이블**
- **목표**: **시간대별 거래량**을 추적하여 마케팅 캠페인 시간을 최적화합니다.
- **주요 컬럼**:
  - `hour` (INT): 시간대 (0~23)
  - `total_volume` (DOUBLE PRECISION): 해당 시간대의 총 거래량

---

## 3️⃣ **Redash에서 실행해야 하는 최종 쿼리**

### 1. **전체 시장 트렌드 분석**

**목표**: 시장 트렌드를 파악하고, **상승/하락하는 코인의 비율**과 **전체 시가총액**을 시각화합니다.

```sql
SELECT 
    timestamp,
    SUM(price * volume) AS total_market_cap,
    (SUM(CASE WHEN price > LAG(price) OVER (PARTITION BY coin_id ORDER BY timestamp) THEN 1 ELSE 0 END) / COUNT(*)) * 100 AS market_up_percentage,
    (SUM(CASE WHEN price < LAG(price) OVER (PARTITION BY coin_id ORDER BY timestamp) THEN 1 ELSE 0 END) / COUNT(*)) * 100 AS market_down_percentage
FROM coin_statistics
GROUP BY timestamp
ORDER BY timestamp;

파이 차트: 상승/하락 비율을 계산하여 파이 차트로 표시
라인 차트: 전체 시가총액 변화를 라인 차트로 표시

### 2. **7일 가격 변동률 상위 5개 코인**
목표: 7일 동안의 가격 변동률을 계산하고 상위 5개 코인을 추출합니다. 이때, IQR 방식으로 거래량의 이상치를 제외합니다.

```sql
복사
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

테이블 차트: 상위 5개 코인의 가격 변동률, 거래량, 시가총액을 표시
IQR 필터링: 거래량이 과도하게 작은 코인 제외
### 3. **시간대별 거래량 변화**
목표: 거래량이 많은 시간대를 추적하여 마케팅 전략을 최적화합니다.

```sql
SELECT 
    EXTRACT(HOUR FROM timestamp) AS hour,
    SUM(volume) AS total_volume
FROM coin_statistics
WHERE timestamp BETWEEN NOW() - INTERVAL '7 days' AND NOW()
GROUP BY hour
ORDER BY hour;
라인 차트: 시간대별 거래량 변화를 표시하여 가장 거래량이 많은 시간대를 파악
4️⃣ 데이터 마트 테이블 관계도
📑 테이블	📋 설명
coin_statistics	실시간 거래 기록 (가격, 시가총액, 거래량 등)
trend_analysis	전체 시장 트렌드 분석 (상승/하락 비율, 시가총액)
top_price_change_7d	7일 가격 변동률 상위 5개 코인 분석 (IQR 필터링 포함)
hourly_volume	시간대별 거래량 분석
🚀 데이터 마트 구축 목표
효율적인 분석을 위한 데이터 집계: 거래량, 시가총액, 가격 변동률 등을 주기적으로 집계하여 대시보드에서 빠르게 조회하고 시각화할 수 있도록 준비합니다.
이상치 필터링: IQR 방식으로 이상치를 제외하고, 과도한 변동을 제거하여 더 정확한 분석을 제공합니다.
시각화 및 대시보드: 집계된 데이터를 Redash나 다른 시각화 도구에서 사용할 수 있도록 저장하여 마케팅팀이 필요한 정보를 실시간으로 확인할 수 있도록 합니다.
📌 결론
원본 테이블 (coin_statistics): 실시간 거래 데이터를 기록합니다.
집계된 테이블:
trend_analysis: 시장 트렌드 분석 (시가총액 및 상승/하락 비율)
top_price_change_7d: 7일 가격 변동률 상위 5개 코인
hourly_volume: 시간대별 거래량 분석
이 구조를 통해 효율적인 데이터 집계와 정확한 분석을 지원하고, Redash 대시보드에서 실시간 데이터 분석이 가능하게 됩니다.

