# 🚀 가상자산 데이터 분석 프로젝트

## 📌 1. 프로젝트 개요
**이 프로젝트는 가상자산 거래소의 데이터를 기반으로 데이터 마트를 구축하고, 이를 분석하여 이상 거래 탐지 및 시장 트렌드 분석을 수행하는 것입니다.**  
**또한, 분석 결과를 Redash를 통해 시각화하여 운영팀이 실무에서 활용할 수 있도록 합니다.**

---

## 📌 2. 분석 목적
### 🎯 **가상자산 거래소 운영팀의 핵심 과제**
1️⃣ **시장 트렌드 모니터링:** 어떤 코인의 거래량이 급등하고 있는가?  
2️⃣ **이상 거래 탐지:** 특정 시간대에 비정상적으로 높은 거래가 발생하는가?  
3️⃣ **Pump & Dump 사기 감지:** 특정 코인이 급등 후 급락하는 패턴이 있는가?  
4️⃣ **매수 vs 매도 패턴 분석:** 현재 시장에서 매수와 매도가 어떻게 이루어지고 있는가?

✅ 이를 위해, 데이터 마트를 구축하고 SQL 분석을 수행하여 운영팀이 즉시 활용할 수 있도록 정리합니다.

---

## 📌 3. 데이터 마트 구축 & 정합성 검증
### ✅ 데이터 마트 테이블 설계
#### `trades` 테이블 (거래 데이터)
```sql
CREATE TABLE trades (
    trade_id SERIAL PRIMARY KEY,
    binance_trade_id BIGINT UNIQUE,
    price DECIMAL(18,8),
    quantity DECIMAL(18,8),
    quote_quantity DECIMAL(18,8),
    trade_time TIMESTAMP,
    is_buyer_maker BOOLEAN
);
```
#### `market_data` 테이블 (시장 데이터)
```sql
CREATE TABLE market_data (
    market_id SERIAL PRIMARY KEY,
    coin_id VARCHAR(50),
    symbol VARCHAR(20),
    current_price DECIMAL(18,8),
    high_24h DECIMAL(18,8),
    low_24h DECIMAL(18,8),
    total_volume DECIMAL(24,8),
    market_cap DECIMAL(24,8),
    last_updated TIMESTAMP
);
```

### ✅ 데이터 정합성 검증
```sql
-- 중복 데이터 확인
SELECT coin_id, COUNT(*) FROM market_data GROUP BY coin_id HAVING COUNT(*) > 1;

-- NULL 값 탐지
SELECT * FROM market_data WHERE current_price IS NULL OR market_cap IS NULL;

-- 이상 데이터 탐지 (비정상적인 가격 데이터)
SELECT * FROM market_data WHERE current_price <= 0 OR market_cap <= 0;
```
✅ **정합성 검증 결과:**  
✅ 중복 없음 / ✅ NULL 없음 / ✅ 이상 데이터 없음 🎉  

---

## 📌 4. 핵심 데이터 분석 & SQL 구현

### **1️⃣ 거래량 급등 코인 분석 (Trading Volume Growth)**
```sql
SELECT symbol, total_volume,
       LAG(total_volume) OVER (ORDER BY last_updated) AS prev_volume,
       ROUND((total_volume - LAG(total_volume) OVER (ORDER BY last_updated)) / NULLIF(LAG(total_volume) OVER (ORDER BY last_updated), 0) * 100, 2) AS volume_growth
FROM market_data
ORDER BY volume_growth DESC
LIMIT 10;
```
📌 **활용:** 🚀 **거래량이 급등한 코인을 실시간으로 감지하여 마케팅 & 신규 투자 기회 제공**  

---

### **2️⃣ 매수 vs 매도 패턴 분석 (Buyer vs Seller Activity)**
```sql
SELECT is_buyer_maker, COUNT(*) AS trade_count, 
       ROUND( COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(), 2) AS trade_percentage
FROM trades
GROUP BY is_buyer_maker;
```
📌 **활용:**
- `is_buyer_maker = TRUE` → 매수 측에서 거래한 횟수  
- `is_buyer_maker = FALSE` → 매도 측에서 거래한 횟수  
- 🚀 **시장 참여자들이 매수/매도를 어느 쪽에서 더 많이 하고 있는지 분석 가능!**  

---

### **3️⃣ Pump & Dump 패턴 감지**
```sql
WITH price_change AS (
    SELECT symbol, 
           last_updated,
           current_price,
           LAG(current_price) OVER (PARTITION BY symbol ORDER BY last_updated) AS prev_price,
           (current_price - LAG(current_price) OVER (PARTITION BY symbol ORDER BY last_updated)) / NULLIF(LAG(current_price) OVER (PARTITION BY symbol ORDER BY last_updated), 0) * 100 AS price_change_pct
    FROM market_data
)
SELECT symbol, last_updated, current_price, price_change_pct
FROM price_change
WHERE price_change_pct > 50 OR price_change_pct < -50  
ORDER BY ABS(price_change_pct) DESC;
```
📌 **활용:** 🚨 **Pump & Dump(펌핑 후 덤핑) 가능성이 높은 코인 감지하여 금융 사기 방지**

---

## 📌 5. Redash를 활용한 데이터 시각화
### **운영팀이 실시간 모니터링할 대시보드 구성**
| **Redash 차트 유형** | **시각화할 분석** |
|-----------------|------------------|
| 📊 **Bar Chart** | 최근 거래량 급등 코인 순위 |
| 📉 **Line Chart** | 특정 코인의 이동 평균 vs 실시간 가격 비교 |
| 🔥 **Heatmap Chart** | 특정 시간대 매수/매도 비율 변화 분석 |
| 🕯 **Candlestick Chart** | Pump & Dump 패턴이 발생한 코인 분석 |

---

## 📌 6. 결론 및 적용 방안
✅ **거래소 운영팀이 실시간으로 거래 데이터 이상 탐지 가능**  
✅ **Wash Trading, Pump & Dump 사기 방지 가능**  
✅ **Redash 대시보드를 활용해 데이터 기반 의사 결정 가능**  
