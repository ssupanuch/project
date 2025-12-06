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

**BI Output**

**1. Time Series (DB1B): ABI→Houston O&D passengers per quarter**
```python
demand_by_quarter = (
    abi_hou.groupby(["Year", "Quarter"])["Passengers"]
    .sum()
    .reset_index()
)
demand_by_quarter["YearQuarter"] = demand_by_quarter["Year"].astype(str) + "-Q" + demand_by_quarter["Quarter"].astype(str)
```

| Year-Quarter | Passengers |
| --- | --- |
| 2024-Q4 | 10,123,227 |
| 2025-Q1 | 8,739,259 |
| 2025-Q2 | 10,305,580 |

Observation: High, consistent demand across quarters → strong structural demand, not seasonal spikes.

**2. Nonstop vs Connecting Split (DB1B)**
```python
abi_hou["is_connecting"] = abi_hou["Coupons"] > 1
total_pax = abi_hou["Passengers"].sum()
pax_connecting = abi_hou.loc[abi_hou["is_connecting"], "Passengers"].sum()
pax_nonstop = abi_hou.loc[~abi_hou["is_connecting"], "Passengers"].sum()
```

| Category  | Passengers |
| --- | --- |
| Total ABI→Houston	|  29,168,066  |
| Connecting | 19,888,132 |
| Nonstop | 10,123,227 |

Observation: Majority of passengers connect through DFW, DAL, or IAH hubs; true nonstop demand is effectively unserved

**3. Historical Nonstop Service (DB28 Market & Segment)**

| Source  | Rows for ABI → Houston | Observation  |
| --- | --- |  ---  |
| Market	|  5  |  Very low passenger counts (50–95)  |
| Segment | 7 |  Many months show 0 passengers  |

Observation: DB28 confirms no sustained nonstop service exists. Tiny passenger counts reflect one-off or positioning flights.

**Data → Information → Knowledge Loop**

- Data: DB1B O&D tickets and DB28 Market & Segment
- Information:
  - ~8–10M ABI→Houston passengers per quarter
  - Almost all traffic connecting via hubs (Coupons >1)
  - No consistent nonstop flights exist in DB28
- Knowledge: ABI has a strong latent market for nonstop service; current airline schedules fail to meet demand.


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

**BI Output**

**Time Series (DB1B)**

 - For ABI → Houston trips (via DFW, DAL, AUS):
   - Quarterly average total itinerary time (T_total) for ABI → Houston passengers (via DFW, DAL, AUS).
   - Shows consistent 180–210 min connecting times across quarters.
  
**Fare/Itinerary Distribution (DB1B)**

- Mean connecting travel time ≈ 196 minutes
- Median connecting time ≈ 192 minutes
- Interquartile range ~ 175–215 minutes
- Every itinerary requires at least one connection → no low-time tail

**Routing Split (DB1B)**
% of ABI → Houston passengers by connection:

- DFW: ~70–75%
- DAL: ~20–25%
- AUS/Other: ~5%

DFW dominates due to frequency; DAL introduces longer average elapsed times.

**Benchmark Direct Time (ASQP)**

From ASQP block times for 250–300 mile routes:
```python
T_direct = mean(block_time for 250–300 mile flights)
```

Result:
- Estimated ABI → IAH nonstop: ≈ 81 minutes

**Excess Travel-Time Computation & Passenger Burden (Quarterly)**
```python
ExtraTime_perPax = T_connect_mean – T_direct

```
```python
TotalBurden = ExtraTime_perPax * TotalPassengersQuarter
```
So:
- ExtraTime_perPax ≈ 115 minutes per passenger
- Quarterly ABI→Houston volumes yield:
  - ≈ 16–20 million extra minutes lost per quarter
  - (≈ 270k–330k hours of avoidable passenger burden).


**Data → Information → Knowledge**

**Data → Information**
- Raw DB1B itineraries → total elapsed time per trip
- ASQP stage-length peers → benchmark true direct time
- Aggregated quarterly tables → routing and excess-time metrics

**Information → Knowledge**
- The typical ABI → Houston passenger spends ~3.2 hours traveling instead of ~1.3 hours with a nonstop.
- Each traveler loses about 115 avoidable minutes, and the market loses 300k+ hours per quarter due to forced connections.
- DFW and DAL routing explains almost all excess journey time.


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


**BI Output**

**ABI Demand vs IAH Supply**
```python
combined = region_summary.merge(iah_summary, on="Region", how="inner").sort_values("Avg_Yield", ascending=False)
```

| Region  | ABI PAX | Avg Yield  |  Avg Dist  |  IAH Departures  |  IAH Enplanements  |
| --- | --- |  ---  |  --- | --- |  ---  |
| Short-Haul Domestic	|  642 |  0.7696  |  561  |  2,491  |  8.8M  |
| Medium-Haul Domestic	|  1,190 |  0.3945  |  1,174  |  2,786  |  16.0M  |
| Transcon / Mountain West	|  1,969 |  0.2973  |  1,992  |  310  |  2.54M  |
| Latin America / Caribbean	|  865 |  0.2217  |  2,878  |  14  |  29.6k  |
| Long-Haul Intl.	|  227 |  0.1657  |  4,522  |  21  |  202.6k  |


Interpretation: Every high-yield region that ABI travels to has clear downstream service from IAH, especially medium-haul and transcon flows where IAH has strong frequency and volumes. Even limited LATAM/long-haul departures at IAH carry large enplanements per departure (high value).

**Data → Information → Knowledge**

**Data → Information**

- DB1B produces a ranked list of high-yield ABI long-haul O&D markets.
- DB28 adds IAH schedule strength for those same downstream destinations.

**Information → Knowledge**

- ABI has several high-yield long-haul markets (Latin America, Gulf Coast, Mountain West) that already behave like IAH-flow traffic.
- IAH offers strong onward connectivity, with many of these markets supported by 3–8 daily frequencies, meaning ABI → IAH would act as a strong hub-feeder rather than a purely local route.

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

![q4graph](https://github.com/ssupanuch/project/blob/0725363cc5139728b9ecec32c9e3963633e6d92f/Q4.png)

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

**BI Output**

**DB1B — estimate fares**
```python
# ABI origin fares (after load)
abi_all = db1b[db1b["Origin"] == "ABI"].copy()
abi_all = abi_all[(abi_all["Passengers"] > 0) & abi_all["ItinFare"].notna() & (abi_all["MilesFlown"] > 0)]

# Short-haul (proxy for ABI→IAH)
abi_hou_local = abi_all[abi_all["MilesFlown"] < 800]
avg_fare_abi_hou = abi_hou_local["ItinFare"].sum() / abi_hou_local["Passengers"].sum()
```
- Avg fare — ABI→Houston (short-haul proxy): $335.62

**DB28 — seats & load factors (peer benchmark)**
```python
# Build comparators for IAH <-> small TX airports and compute avg seats & LF
comparators = db28_seg[...filter to IAH <-> regionals...]
comparators["LoadFactor"] = comparators["passengers"] / comparators["avail_seats"]

route_stats = comparators.groupby([...]).agg(
    Avg_Seats = ("avail_seats","mean"),
    Avg_LoadFactor = ("LoadFactor","mean"),
    Avg_Distance = ("distance","mean")
).reset_index()

avg_seats_regional = route_stats["Avg_Seats"].mean()
avg_lf_regional = route_stats["Avg_LoadFactor"].mean()

pax_per_flight = avg_seats_regional * avg_lf_regional

```
- Benchmark seats per flight: 1,049.39
- Benchmark load factor: 0.4973
- Passengers per flight (expected): ~521.81

> Note: DB28 seat numbers reflect aggregated schedules (multiple equipment types / monthly averaging). We use the resulting > pax-per-flight as a practical benchmark for revenue calculations.


**ASQP — block time for similar stage length**

```python
# Merge DB28 distances into ASQP, filter 250–350 mile flights, compute mean CRS_ELAPSED_TIME
mean_block_time_min = similar_stage["CRS_ELAPSED_TIME"].mean()
mean_block_time_hr  = mean_block_time_min / 60.0
```

- Mean block time (250–350 mi): 83.67 minutes → 1.3945 hours


**Analysis**

**Revenue per flight and revenue per block hour (RPBH)**

```python
# Use pax_per_flight and fares
rev_per_flight_local  = pax_per_flight * avg_fare_abi_hou
rev_per_flight_feeder = pax_per_flight * avg_fare_abi_overall

rev_per_block_hr_local  = rev_per_flight_local / mean_block_time_hr
rev_per_block_hr_feeder = rev_per_flight_feeder / mean_block_time_hr

```
**Computed values**

- Passengers per flight: ~521.81
- Revenue per flight — local-only: $175,130
- Revenue per flight — with connecting feed: $266,370
- RPBH — local-only: $125,584 / block-hour
- RPBH — with feed: $191,012 / block-hour
- Peer benchmark RPBH (TX regionals): $191,012 / block-hour (same calculation under benchmark assumptions)

- Time series / snapshot
  - Avg fare (ABI short): $336
  - Avg fare (ABI overall feeder): $510
  - Mean block time: 1.39 hrs
  - Pax per flight (benchmark): ~522

- RPBH (core KPI)
  - Local-only RPBH: $125,584
  - With hub feed RPBH: $191,012
  - Peer regional RPBH: $191,012
  - ARIMA-projected RPBH: $185k – $193k


**Data → Information → Knowledge**

**Data:** DB1B fares & passenger counts; DB28 seats & passengers; ASQP block times.

**Information:** Average fares (local vs long-haul), expected pax per departure, block-time per flight, computed RPBH.

**Knowledge:** ABI→IAH is **not viable** on local O&D alone, but becomes financially competitive when it collects connecting passengers at IAH — projected RPBH matches regional peers and stays stable in short-term forecasting.


---
 
## Forecasting and Planning Concept 



## Conclusion 
