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

