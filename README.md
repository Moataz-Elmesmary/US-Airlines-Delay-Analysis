# ✈️ US Airlines Delay Analysis - Power BI Project
<img src="https://github.com/user-attachments/assets/bdeab8a7-cbab-48e5-821b-9b40ad2991a4" width="700"/>


## 📜 Project Overview

This Power BI project provides a comprehensive analysis of flight delays within the United States. Utilizing a detailed dataset from the U.S. Department of Transportation, the project aims to uncover patterns, identify root causes, and highlight key performance indicators (KPIs) related to airline and airport efficiency. The dashboards offer actionable insights for airlines, passengers, and regulatory bodies seeking to understand and mitigate flight disruptions.

## 📊 Data Source

The dataset used in this project is sourced from:
* **Original Data Source:** U.S. Department of Transportation's (DOT) Bureau of Transportation Statistics (BTS).
* **Kaggle Dataset:** [Airlines Delay Causes](https://www.kaggle.com/datasets/giovamata/airlinedelaycauses?select=DelayedFlights.csv)
* **Specific File:** `DelayedFlights.csv`

The data tracks domestic flights operated by large air carriers, including on-time performance, delays, cancellations, and diversions, with detailed causes of delays starting from June 2003.

## 🎯 Problem Statement & Project Goal

**Problem:** Flight delays are a significant inconvenience for passengers and a costly operational challenge for airlines. Understanding the underlying factors and quantifying their impact is crucial for improvement.

**Goal:**
* To visualize overall flight performance and identify key disruption metrics.
* To deep-dive into the causes of delays (e.g., carrier, weather, NAS) and their severity.
* To analyze the performance of individual airlines and airports regarding delays and cancellations.
* To identify seasonal and time-based patterns in flight delays.
* To provide an interactive Power BI dashboard solution that allows stakeholders to explore data and derive actionable insights.

## 🚀 Data Acquisition & Overview

The data was acquired directly from the `DelayedFlights.csv` file. It's a single, large CSV file containing millions of rows, each representing a domestic flight. Key columns include flight identification, times (scheduled and actual), delay minutes by cause, and cancellation/diversion flags.

## Data Cleaning & Transformation (Power Query)

## Data Modeling (Relationships)

The core of the Power BI project's analytical power lies in its data model, which follows a **Star Schema** design.

* **Fact Table:** `DelayedFlights` (The main table containing individual flight records and numerical measures).
* **Dimension Tables:** `Dim_Airlines`, `Dim_Airports`, `Dim_Date` (Lookup tables containing descriptive attributes related to flights).

### **Relationships Created:**

| From Table       | From Column         | To Table         | To Column         | Cardinality | Cross Filter Direction | Status     | Notes                                                                                                                                                                                                                             |
| :--------------- | :------------------ | :--------------- | :---------------- | :---------- | :--------------------- | :--------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Dim_Airlines`   | `AirlineCode`       | `DelayedFlights` | `UniqueCarrier`   | One-to-Many | Single                 | Active     | Filters `DelayedFlights` by `AirlineCode` (e.g., when selecting an airline name from a slicer).                                                                                                                                         |
| `Dim_Airports`   | `AirportCode`       | `DelayedFlights` | `Origin`          | One-to-Many | Single                 | **Inactive** | Filters `DelayedFlights` based on the origin airport. Requires `USERELATIONSHIP` in DAX measures for specific "Origin" based calculations (e.g., `Total Delayed Flights by Origin Airport`).                                         |
| `Dim_Airports`   | `AirportCode`       | `DelayedFlights` | `Dest`            | One-to-Many | Single                 | **Inactive** | Filters `DelayedFlights` based on the destination airport. Requires `USERELATIONSHIP` in DAX measures for specific "Destination" based calculations (e.g., `Total Delayed Flights by Destination Airport`).                         |
| `Dim_Date`       | `Date`              | `DelayedFlights` | `FlightDate`      | One-to-Many | Single                 | Active     | Filters `DelayedFlights` by various date attributes (Year, Month, Day) using `Dim_Date` columns. This relationship enables Time Intelligence.                                                                                           |
<img src="https://github.com/user-attachments/assets/bdab8ed7-e606-45d3-b117-3d9eb03bd523" width="700"/>


## 📈 Key Performance Indicators (KPIs) & DAX Measures

The following DAX measures were created to quantify flight performance and delay characteristics:

```dax
-- 1. Total Flights & Status
Total Flights = COUNTROWS('DelayedFlights')

Total Delayed Flights =
CALCULATE(
    COUNTROWS('DelayedFlights'),
    OR(
        'DelayedFlights'[ArrDelay] > 0,
        'DelayedFlights'[DepDelay] > 0
    )
)

Total Cancelled Flights =
CALCULATE(
    COUNTROWS('DelayedFlights'),
    'DelayedFlights'[Cancelled] = 1
)

Total Diverted Flights =
CALCULATE(
    COUNTROWS('DelayedFlights'),
    'DelayedFlights'[Diverted] = 1
)

Total On-Time Flights =
[Total Flights] - [Total Delayed Flights] - [Total Cancelled Flights] - [Total Diverted Flights]

-- 2. Performance Rates
Delay Rate (%) =
DIVIDE(
    [Total Delayed Flights],
    [Total Flights],
    0
)

Cancellation Rate (%) =
DIVIDE(
    [Total Cancelled Flights],
    [Total Flights],
    0
)

On-Time Performance (%) =
DIVIDE(
    [Total On-Time Flights],
    [Total Flights],
    0
)

-- 3. Delay Durations (Minutes)
Average Arrival Delay (Min) =
AVERAGEX(
    FILTER('DelayedFlights', 'DelayedFlights'[ArrDelay] > 0),
    'DelayedFlights'[ArrDelay]
)

Average Departure Delay (Min) =
AVERAGEX(
    FILTER('DelayedFlights', 'DelayedFlights'[DepDelay] > 0),
    'DelayedFlights'[DepDelay]
)

Total Carrier Delay (Min) = SUM('DelayedFlights'[CarrierDelay])
Total Weather Delay (Min) = SUM('DelayedFlights'[WeatherDelay])
Total NAS Delay (Min) = SUM('DelayedFlights'[NASDelay])
Total Security Delay (Min) = SUM('DelayedFlights'[SecurityDelay])
Total Late Aircraft Delay (Min) = SUM('DelayedFlights'[LateAircraftDelay])

Total Delay Minutes (All Causes) =
SUM(
    'DelayedFlights'[CarrierDelay] +
    'DelayedFlights'[WeatherDelay] +
    'DelayedFlights'[NASDelay] +
    'DelayedFlights'[SecurityDelay] +
    'DelayedFlights'[LateAircraftDelay]
)

Average Delay Duration (All Causes - Min) =
DIVIDE(
    [Total Delay Minutes (All Causes)],
    [Total Delayed Flights],
    0
)

-- 4. Measures using inactive relationships (for Airport analysis)
Total Delayed Flights by Origin Airport =
CALCULATE (
    [Total Delayed Flights],
    USERELATIONSHIP ( Dim_Airports[AirportCode], DelayedFlights[Origin] )
)

Total Delayed Flights by Destination Airport =
CALCULATE (
    [Total Delayed Flights],
    USERELATIONHIP ( Dim_Airports[AirportCode], DelayedFlights[Dest] )
)
```
<img src="https://github.com/user-attachments/assets/3e2769bc-f5a6-4a92-9e41-dcde3e18feb7" width="700"/>

