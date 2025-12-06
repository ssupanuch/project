# Case Study: Should Southwest Launch ABI-HOU/IAH?

> BI = Business Intelligence  
> Data Analytics & Decision Science = Data Analytics & Decision Science  
> ML = Machine Learning  

---
## Introduction

Our chosen Texas city pair is Abilene to Houston. 

This new direct route will be operated by Southwest Airlines. Southwest Airlines sees the potential growth in this route because Abilene has a mix of business (agriculture, healthcare, energy, and manufacturing) and leisure (universities and smaller cities to travel). Abilene is an easier connection for central Texas. Instead of driving 3 hours to DFW and dealing with chaos to fly, they can drive to Abilene airport, fly to Houston, and then connect to anywhere. This service route is not about tourist demand but a steady and reliable traffic that feeds to a major hub.

**Five Questions:** 

1. Is There Suppressed ABI → Houston Demand Due to Lack of Nonstop Service?
2. How Much Extra Time and Burden Do ABI Passengers Endure Today Without a Nonstop?
3. What High-Yield Connecting Markets Would ABI Gain Through IAH?
4. What Reliability Environment Will ABI–IAH Enter?
5. Can ABI–IAH Produce Competitive Revenue per Block Hour?


## BI / Data Overview

Available data via aviation_core:

- **DB28** — segment & market volumes, seats, load factors on real routes.
- **DB1B** — 10% ticket sample with O&D and fares.
- **ASQP** — flight-level on-time and delay statistics.

Available tools:

- **BI** — 
- **DADS** — 
- **ARIMA** — 
- **XGBoost** — 

## Integrated Questions 

### Q1 — Is There Suppressed ABI → Houston Demand Due to Lack of Nonstop Service?

**Datasets**: DB1B + DB28

**Purpose**: Establish underlying O&D demand that is hidden because all travel is via connections.

**Computation**

- DB1B:
  - Count all ABI ↔ IAH/HOU passengers, regardless of routing.
  - Split into connecting vs (nonexistent) nonstop:
       - PAX_CONNECTING = sum(DB1B pax where connecting)
       - PAX_NONSTOP ≈ 0

- DB28:
  - Confirm that no ABI–Houston nonstop exists in the historical schedule.
  
---

### Q2 — How Much Extra Time and Burden Do ABI Passengers Endure Today Without a Nonstop?

**Datasets**: DB1B + ASQP

**Purpose**: Demonstrate the passenger pain point that a nonstop ABI–IAH would fix.

**Computation**

- DB1B:
  - For ABI → Houston trips (via DFW, DAL, AUS):
       - Extract total itinerary elapsed time (T_total = departure to arrival).

- ASQP:
  - Get block times for flights of similar stage length (≈ 280 miles).
  - Compute what ABI → IAH would take as a direct:
       - T_direct ≈ mean(ASQP block times for 250–300 mile routes)

---

### Q3 — What High-Yield Connecting Markets Would ABI Gain Through IAH?

**Datasets**: DB1B + DB28

**Purpose**: Show that ABI → IAH is a hub-feeder route worth more than its local O&D.

**Computation**

- DB1B:
  - Rank ABI → long-haul O&D markets by yield (revenue per passenger-mile).
  - Identify which naturally connect via Houston (Latin America, Gulf Coast, Mountain West, etc.).

- DB28:
  - Identify the frequency of departures from IAH to those exact connecting markets.

---

### Q4 — What Reliability Environment Will ABI–IAH Enter?

**Datasets**: DB28 + ASQP

**Purpose**: Evaluate operational feasibility and connection protection.

**Computation**

- DB28:
  - Identify the IAH connection banks ABI would feed (morning, midday, evening).

- ASQP:
  - Calculate the reliability of existing IAH feeder flights:
       - Mean ARR_DELAY
       - 90th percentile delay
       - Ground/Taxi times
       - Weather delay frequencies

**Analysis**

Based on data from August 2024 to June 2025, ABI→IAH will enter a moderately reliable environment. Most months show low to moderate arrival delays, supporting connection protection at IAH. The most reliable months are Sep–Nov 2024, with high flight frequency and minimal delays. Some months (Apr–Jun 2025) exhibit higher 90th percentile delays (18–25 minutes) and moderate departures, increasing the risk that passengers could miss tight connections. Overall, the route is operationally feasible, but planning should consider these higher-risk periods when allocating connection buffers.

---

### Q5 — Can ABI–IAH Produce Competitive Revenue per Block Hour?

**Datasets**: DB1B + DB28 + ASQP 

**Purpose**: Tie demand, reliability, and yield into a financial viability question.

**Computation**

- DB1B:
     - Estimate average fare for ABI → Houston and ABI → long-hauls.
- DB28:
  - Get typical seats and load factors for comparable Texas regional routes.
- ASQP:
  - Obtain actual block times for similar stage length operations.

---
 
## Forecasting and Planning Concept 



## Conclusion 
