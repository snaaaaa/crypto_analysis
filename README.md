## 📌 1. 데이터 분석의 목적  

### 🎯 가상자산 거래소 운영팀의 핵심 과제  

1️⃣ **시장 트렌드 모니터링**: 어떤 코인의 거래량이 급등하고 있는가?  
3️⃣ **Pump & Dump 사기 감지**: 특정 코인이 급등 후 급락하는 패턴이 있는가?  

✅ 이를 위해, **데이터 마트를 구축하고 SQL 분석을 수행**하여 운영팀이 즉시 활용할 수 있도록 정리합니다.

---

## 📌 2. 데이터 정합성 검증  

### ✅ 데이터 정합성 검증 과정  

#### 🔍 중복 데이터 확인  
`SELECT coin_id, COUNT(*) FROM market_data GROUP BY coin_id HAVING COUNT(*) > 1;`

#### 🔍 NULL 값 탐지  
`SELECT * FROM market_data WHERE current_price IS NULL OR market_cap IS NULL;`

#### 🔍 이상 데이터 탐지 (비정상적인 가격 데이터)  
`SELECT * FROM market_data WHERE current_price <= 0 OR market_cap <= 0;`

### ✅ 정합성 검증 결과:  
- ✅ **중복 없음**  
- ✅ **NULL 없음**  
- ✅ **이상 데이터 없음** 🎉  

---

## 📌 3. 핵심 데이터 분석  

### 📌 3-1. Pump & Dump 사기 감지  

### 🎯 분석 목적  
- **Pump & Dump 사기**는 특정 코인이 급등 후 급락하는 패턴을 보일 때 발생하는 시장의 비정상적인 움직임을 의미합니다.  
- 이 패턴을 감지하여 사기를 방지하고, 투자자에게 유용한 실시간 정보를 제공합니다.

### 🔍 분석 방법  
1. **코인 거래량 급등과 급락 시점 분석**  
   급등 후 급락하는 패턴을 감지하기 위해, 거래량과 가격 변동을 시간대별로 분석합니다. 급격한 가격 변화가 일어난 시점을 찾아냅니다.  
   
2. **이상적인 패턴을 찾기 위한 규칙 설정**  
   - 거래량이 급격하게 증가한 후, 가격이 급락하는 패턴을 찾습니다.
   - 일정 시간 이내에 급격한 상승과 하락을 동시에 나타내는 코인을 'pump & dump'으로 판단합니다.

### ✅ 결과  
- **이상 거래 감지**: 특정 코인의 거래량 급등 후 급락 패턴을 정확히 감지하여, 시장에서의 비정상적인 활동을 파악할 수 있습니다.  
- **경고 시스템**: 실시간으로 'pump & dump' 패턴을 감지하고 경고 시스템을 구축하여, 투자자에게 위험을 알릴 수 있는 기회를 제공합니다.

---

### 📌 3-2. 시장 트렌드 모니터링  

### 🎯 분석 목적  
- **시장 트렌드 모니터링**은 특정 코인의 거래량 변화와 시장 트렌드를 파악하여, 투자자에게 유망한 투자처를 안내할 수 있도록 돕습니다.  
- 다양한 코인의 시장 상황을 종합적으로 분석하여 트렌드를 예측합니다.

### 🔍 분석 방법  
1. **거래량 급등 코인 분석**  
   - 시장에서 급격히 거래량이 증가하는 코인을 실시간으로 분석하여, 현재 투자자들이 주목하는 코인들을 파악합니다.  
   - 거래량 급증 시점과 함께 상승률을 고려하여, 유망한 코인을 선별합니다.

2. **거래량 증가율 및 시장 점유율 분석**  
   - 거래량 변화율을 기준으로 코인들의 트렌드를 예측하고, 시장에서 차지하는 점유율 변화를 모니터링합니다.

### 1️⃣ 거래량 급등 코인 분석 (Trading Volume Growth)  
`WITH daily_volume AS (  
    SELECT DATE(last_updated) AS date, symbol,  
           SUM(total_volume) AS total_volume  
    FROM market_data  
    GROUP BY DATE(last_updated), symbol  
),  
volume_change AS (  
    SELECT date, symbol, total_volume,  
           LAG(total_volume) OVER (PARTITION BY symbol ORDER BY date) AS prev_volume,  
           ROUND((total_volume - LAG(total_volume) OVER (PARTITION BY symbol ORDER BY date)) /  
                 NULLIF(LAG(total_volume) OVER (PARTITION BY symbol ORDER BY date), 0) * 100, 2) AS volume_growth  
    FROM daily_volume  
)  
SELECT date, symbol, total_volume, prev_volume, volume_growth  
FROM volume_change  
WHERE prev_volume IS NOT NULL  
ORDER BY volume_growth DESC  
LIMIT 10;`

### 2️⃣ 거래량 급등 필터링 & 최적화  

#### ✅ 데이터 분석 결과  
단순 거래량 증가율(%)만 보면 전체 시장 상황을 반영하지 않을 수 있음.  

**예제:**  
- 거래량이 원래 낮았던 코인이 소폭 증가해도 높은 증가율로 보일 수 있음.  
- BTC처럼 원래 거래량이 높은 코인은 상대적으로 변화율이 낮을 가능성이 큼.  

✅ **따라서, 거래량 변화 금액(절대값)도 함께 고려해야 더 Insightful 함.**  

---

### 🚀 최적의 필터링 기준  
1️⃣ **거래량 변화 금액이 1.2억 이상인 경우만 유지**  
   ✅ 의미 있는 변동이 포함되면서도 너무 많은 코인을 걸러내지 않음.  
   ✅ 대형 코인만 남는 문제를 방지하면서, 실제 상승 트렌드를 반영함.  

2️⃣ **거래량 증가율은 높은 순서로 정렬**  
   ✅ 증가율이 높은 코인을 투자자들에게 강조할 수 있도록 필터링하지 않고 정렬만 수행.  

---

### ✅ 최종 SQL 필터링 적용  
`SELECT *  
FROM volume_change  
WHERE (total_volume - prev_volume) > 120000000  -- 최소 거래량 변화 금액 1.2억 이상  
ORDER BY volume_growth DESC;`

### ✅ 결과  
- **상위 코인 선별**: 급격히 거래량이 증가한 코인을 모니터링하여, 투자자들에게 시장에서 주목받고 있는 유망 코인을 추천할 수 있습니다.  
- **시장 동향 파악**: 시장의 주요 트렌드를 실시간으로 파악하여, 빠르게 변동하는 시장에서 유리한 포지션을 선점할 수 있습니다.

---
