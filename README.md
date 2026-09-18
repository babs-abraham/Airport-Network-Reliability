# Airline Operations & Reliability Analysis

**Power BI | Power Query | DAX | Data Modelling**

*U.S. BTS Flight Performance Data | January 2024 – December 2025*

---

## 1. Project Overview

This project analyses airline operational reliability using monthly U.S. Bureau of Transportation Statistics (BTS) flight-performance data for 2024 and 2025. The objective was to transform fragmented monthly files into a structured Power BI model that answers operational and management questions around flight reliability, delays, cancellations, diversions, carriers, airports, and routes.

The project demonstrates an end-to-end BI workflow: data acquisition, Power Query staging and transformation, data-quality troubleshooting, dimensional modelling, DAX development, validation, dashboard design, and business interpretation.

## 2. Business Problem

Airline reliability cannot be understood from a single KPI. Management needs to know where performance deteriorates, which carriers and routes create the greatest risk, whether delays originate at departure or persist through arrival, and how reliability changes over time.

**Executive question:** How reliable is the airline network, and when/why does performance deteriorate?

## 3. Business Questions

- How many flights were operated?
- What percentage arrived on time?
- How often were flights cancelled or diverted?
- How has reliability changed over time?
- What are the major causes of delay?
- Are particular time periods consistently worse?
- How does departure delay compare with arrival delay?
- Which airlines have the lowest on-time performance?
- Which routes have the highest delay rates?
- Which airports have the weakest departure/arrival performance?
- Do high-volume routes also have poor reliability?
- Which delays are carrier-controlled versus external?
- Which operations deserve further investigation?

## 4. Dataset & Scope

- **Source:** U.S. Bureau of Transportation Statistics (BTS) Reporting Carrier On-Time Performance data.
- **Coverage:** monthly files from January 2024 through December 2025.
- **Final model scale:** approximately 14.1 million flight records.
- **Grain:** flight-level operational records.
- **Core entities:** flights, airlines, airports, cancellation codes, and dates.
- **Analysis:** reliability, delays, cancellations, diversions, carriers, airports, routes, and time trends.

## 5. Data Preparation & Power Query

Monthly files were extracted and consolidated in Power Query. A staging layer was created before separating the data into fact and dimension tables. This provided a repeatable transformation workflow and made data-quality issues easier to investigate.

- Consolidated monthly 2024–2025 source files.
- Created staging data before modelling.
- Standardised data types and structures across files.
- Checked duplicates and investigated transformation errors.
- Created `RouteKey` for route-level analysis.
- Created `DelayMinutePerFlight` to normalise delay burden.
- Created `DelayCategory` for severity analysis.
- Validated relationships, calculations, and filter behaviour.
- Troubleshot loading and refresh issues caused by the large dataset.

## 6. Data Model

The model follows a star-schema approach with `Fact_flights` at the centre.

- **Fact_flights** — flight-level operational measures and engineered analytical fields.
- **Dim_Airline** — carrier attributes.
- **Dim_Airport** — airport attributes.
- **Dim_CancellationCode** — cancellation categories.
- **Date table** — year, month, quarter, and time intelligence.
- **RouteKey** — origin/destination route analysis.

## 7. Important DAX Measures

A dedicated measures table was created to keep business logic reusable and consistent. Key measures included:

```dax
Total Flights := COUNTROWS(Fact_flights)

Completed Flights := CALCULATE([Total Flights], Fact_flights[CANCELLED] = 0)

Cancelled Flights := CALCULATE([Total Flights], Fact_flights[CANCELLED] = 1)

Cancellation % := DIVIDE([Cancelled Flights], [Total Flights], 0)

Diverted Flights := CALCULATE([Total Flights], Fact_flights[DIVERTED] = 1)

Diverted % := DIVIDE([Diverted Flights], [Total Flights], 0)

Average Arrival Delay := AVERAGE(Fact_flights[ARR_DELAY])

Average Departure Delay := AVERAGE(Fact_flights[DEP_DELAY])

Delayed Arrival := CALCULATE([Total Flights], Fact_flights[ARR_DELAY] > 0)

Delayed Departure := CALCULATE([Total Flights], Fact_flights[DEP_DELAY] > 0)

Arrival Delay Rate := DIVIDE([Delayed Arrival], [Eligible Arrival Flights], 0)

Departure Delay Rate := DIVIDE([Delayed Departure], [Eligible Departure Flights], 0)

Delay Minutes per Flight := DIVIDE(SUM(Fact_flights[ARR_DELAY]), [Eligible Arrival Flights], 0)

PY Flights := CALCULATE([Total Flights], SAMEPERIODLASTYEAR('Date'[Date]))

PY On-Time Arrival % := CALCULATE([Arrival On Time %], SAMEPERIODLASTYEAR('Date'[Date]))

PY Average Arrival Delay := CALCULATE([Average Arrival Delay], SAMEPERIODLASTYEAR('Date'[Date]))

Average Delay YoY Change := [Average Arrival Delay] - [PY Average Arrival Delay]

Cancellation YoY Change := [Cancellation %] - [PY Cancellation %]

Severe Delay % := DIVIDE([Severe Delays], [Eligible Delay Flights], 0)

Route Reliability Gap := [Route On-Time %] - [Network Benchmark]
```

Additional model logic included `RouteEligible`, `High Risk Route`, `DelayCategory`, `OnTime Arrival Status`, `CarrierGapLabel`, `Top Route by Carrier`, and year-over-year flight/cancellation/diversion measures.

## 8. Dashboard Structure

### Page 1 — Network Reliability Overview

- Flight volume and network scale.
- Overall on-time arrival performance.
- Cancellation and diversion frequency.
- Delay categories and delay burden.
- Time-based reliability trends.
- Departure versus arrival delay comparison.
- Interactive airline and other dimension filters.
- Dynamic page title that changes with the selected airline.

### Page 2 — Carrier & Route Performance

- Carrier-level on-time performance.
- Route delay and reliability analysis.
- Airport departure and arrival performance.
- High-volume routes with weak reliability.
- Carrier-controlled versus external delay analysis.
- High-risk operations requiring investigation.

## 9. Analysis & Business Findings

The dashboard was designed to turn operational metrics into management insight. The main findings supported by the analysis framework are:

- **Reliability varies across carriers and routes.** A network-level average can conceal substantial differences between individual carriers and routes. Carrier and route comparisons are therefore important for targeted operational investigation.
- **Departure and arrival performance tell different stories.** Comparing departure and arrival delay rates helps identify whether delays originate before departure, persist through the journey, or worsen by arrival.
- **High-volume routes can represent greater business impact.** A route does not need to have the worst percentage delay rate to be operationally important. Combining flight volume with reliability helps prioritise investigation.
- **Delay frequency and delay severity should be separated.** Delay rate alone does not show the operational burden of very long delays. Severe-delay percentage and delay-minutes-per-flight add useful context.
- **Time trends reveal deterioration that averages can hide.** Monthly and year-over-year measures allow management to identify periods of worsening or improving reliability.
- **Cancellations and diversions require separate treatment.** A diverted flight is not automatically a cancelled flight. The completed-flight logic therefore counts non-cancelled diverted flights as completed.
- **Data quality directly affects business confidence.** The project required checks for duplicates, data types, relationships, transformation errors, and calculation behaviour before the dashboard could be trusted.

> **Note:** Carrier, airport, and route rankings should be interpreted using the project's eligibility and volume rules. The dashboard identifies patterns and areas for investigation; it does not by itself prove causation.

## 10. Recommendations

- Prioritise operational investigations using reliability gap, flight volume, and delay severity together.
- Review consistently underperforming carriers and routes for recurring causes.
- Separate carrier-controlled delays from external factors when assigning corrective ownership.
- Use monthly and YoY trends to identify deterioration early.
- Investigate high-risk routes only after applying appropriate volume/eligibility thresholds.
- Monitor departure and arrival performance together to identify where delays originate.
- Maintain a KPI dictionary with agreed definitions before using the dashboard for formal operational reporting.
- Future versions could add targets, forecasting, automated refresh monitoring, and more detailed operational segmentation.

## 11. Limitations & Assumptions

- The analysis depends on the definitions and fields supplied by BTS.
- Monthly files required consolidation and standardisation before analysis.
- The large data volume created Power Query loading and troubleshooting challenges.
- Low-volume routes can produce unstable percentages, so route eligibility matters.
- The analysis identifies patterns but does not establish causation.
- Delay thresholds and categories are analytical definitions for this portfolio project and should be aligned with an organisation's KPI policy before operational use.

## 12. Skills Demonstrated

- **Power BI:** interactive dashboards, KPI design, filtering, and data storytelling.
- **Power Query:** extraction, staging, transformation, append/merge, cleaning, and troubleshooting.
- **DAX:** reusable measures, ratios, time intelligence, YoY analysis, and analytical logic.
- **Data Modelling:** star schema, fact/dimension tables, relationships, and date tables.
- **Data Quality:** duplicate checks, data-type validation, relationship testing, and calculation validation.
- **Business Analysis:** translating operational questions into metrics, findings, and recommendations.
- **Problem Solving:** debugging transformation, loading, and refresh issues in a large-volume environment.

## 13. Portfolio Outcome

This project demonstrates the ability to take a large, fragmented operational dataset and turn it into a structured analytical product. It shows the complete BI workflow from source data and transformation through modelling, DAX, validation, visualisation, and business recommendations.

## 14. Suggested Repository Structure

```
airline-operations-reliability/
├── README.md
├── PowerBI/
│   └── Airline_Operations_Reliability.pbix
├── Data/
│   └── README.md
├── Images/
│   ├── overview-dashboard.png
│   ├── reliability-dashboard.png
│   ├── data-model.png
│   ├── fact-table.png
│   ├── dimension-airline.png
│   ├── dimension-airport.png
│   └── source-data.png
└── Documentation/
    └── Airline_Operations_Reliability_Project.docx
```

## 15. Conclusion

The Airline Operations & Reliability Analysis project demonstrates practical business intelligence capability using Power BI, Power Query, and DAX. It combines data preparation, modelling, analytical calculation, and business storytelling to identify where airline reliability is strongest, where it deteriorates, and which areas deserve further investigation.
