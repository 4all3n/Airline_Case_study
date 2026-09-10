# ASG Airlines: End-to-End Data Engineering Case Study

Hi! This repository contains my submission for the **ASG Airlines Data Engineering Case Study** (NeoStats assessment).

In this project, I took raw, messy operational data from an Excel file (`UseCase - Airlines.xlsx`), cleaned and standardized it using Python and Pandas, protected sensitive passenger personal information (PII), modeled the data into a Star Schema, and calculated business KPIs.

---

## Project Overview & Approach

The project is divided into three main phases inside [`Data_Cleaning.ipynb`](Data_Cleaning.ipynb):

1. **Phase 1: Ingestion & Inspection**  
   Loaded all 4 sheets (`flights`, `bookings`, `passengers`, `payments`) and audited shapes, data types, missing values, and duplicate rows.

2. **Phase 2: Data Cleaning & PII Masking**  
   - Restored missing/UNKNOWN airline names using flight ID prefix mapping (`AI`, `6F`, `SJ`, `UK`).
   - Removed duplicate flight and passenger records.
   - Calculated flight duration in minutes (including handling 124 overnight flights via full timestamp subtraction).
   - Removed a corrupted flight that had a negative duration (-1140 minutes).
   - Standardized booking statuses (converted `INVALID` to `CANCELLED`).
   - Flagged missing/invalid payment amounts with `is_amount_missing = True` instead of inventing fake revenue.
   - Protected passenger PII (SHA-256 hashing for Aadhaar and Passport, masked emails and phone numbers, extracted birth year).

3. **Phase 3: Data Modelling & Business KPIs**  
   - Modeled the tables into a **Star Schema** (`dim_flights`, `dim_passengers`, `dim_payments`, `fact_bookings`).
   - Exported clean datasets to `cleaned_data/`.
   - Computed and plotted 7 key business KPIs (the 4 mandatory ones + 3 important commercial ones).

---

## How to Run the Project

I built and tested this project using **Miniconda (Python 3.11)** on Linux.

### 1. Clone the repo and navigate to the folder:
```bash
git clone <your-repo-link>
cd "Airlines Use Case"
```

### 2. Set up the environment:
```bash
conda create -n data_env python=3.11 -y
conda activate data_env
pip install pandas openpyxl matplotlib seaborn python-docx
```

### 3. Open and run the notebook:
```bash
jupyter notebook Data_Cleaning.ipynb
```
*All 49 cells run end-to-end and already contain the executed outputs and charts.*

---

## Repository Structure

```
Airlines Use Case/
│
├── UseCase - Airlines.xlsx                    # Raw input dataset (4 sheets)
├── Use_Case_Airlines Intructions.docx         # Original assignment instructions
├── Data_Cleaning.ipynb                        # Main pipeline notebook (all phases executed)
├── ASG_Airlines_Data_Engineering_Documentation.docx  # Detailed documentation report
├── README.md                                  # You are here!
│
├── cleaned_data/                              # Cleaned CSV files
│   ├── flights_cleaned.csv                    # 1,003 clean flight rows
│   ├── bookings_cleaned.csv                   # 1,000 clean booking rows
│   ├── passengers_masked.csv                  # 1,000 unique passengers (PII protected)
│   ├── payments_cleaned.csv                   # 1,000 payment entries (78 flagged)
│   └── analytical_table.csv                   # 1,363 joined rows for BI reporting
│
└── visuals/                                   # All charts and diagrams (extracted from notebook)
    ├── architecture_diagram.png               # Pipeline architecture
    ├── data_flow_diagram.png                  # Level-1 Data Flow Diagram
    ├── data_model_diagram.png                 # Star Schema diagram
    ├── kpi1_avg_duration_airline.png          # KPI 1: Average Flight Duration
    ├── kpi2_top_routes_traffic.png            # KPI 2: Top 10 Busiest Routes
    ├── kpi3_flight_duration_delays.png        # KPI 3: Duration Distribution & Delays
    ├── kpi4_airline_market_share.png          # KPI 4: Flight Distribution by Airline
    ├── kpi5_booking_status_breakdown.png      # KPI 5: Booking Status Breakdown
    ├── kpi6_revenue_by_airline.png            # KPI 6: Total Revenue by Airline
    └── kpi7_top_revenue_routes.png            # KPI 7: Top 5 Revenue Routes
```

---

## Summary of Data Quality Issues & How I Fixed Them

| Sheet | Issues Found in Raw Data | How I Cleaned It | Final Count |
| :--- | :--- | :--- | :--- |
| **`flights`** | 41 nulls and 31 'UNKNOWN' in `airline`<br>16 duplicate flight IDs<br>Duration had Excel formula `=F2-E2`<br>1 flight had negative duration (-1140 min) | • Inferred airline from prefix: `AI` = Air India, `6F` = IndiGo, `SJ` = SpiceJet, `UK` = Vistara<br>• Dropped 16 duplicate flights<br>• Calculated duration in minutes from arrival and departure times<br>• Filtered out the negative duration anomaly | **1,003 clean flights**<br>(30 to 300 min duration) |
| **`bookings`** | 45 null statuses, 30 'INVALID' statuses<br>Passport numbers in plain text | • Dropped trailing blank rows<br>• Replaced 'INVALID' with 'CANCELLED'<br>• Filled null statuses with 'UNKNOWN'<br>• Hashed passport with SHA-256 | **1,000 clean bookings**<br>(344 Cancelled, 320 Confirmed, 291 Pending, 45 Unknown) |
| **`passengers`** | 36 duplicate passenger IDs (39 rows)<br>10 missing last names (merged into first name)<br>Aadhaar, phone, email in plain text | • Sorted by `last_name` to keep complete records<br>• Dropped duplicates by `passenger_id`<br>• Hashed Aadhaar (SHA-256)<br>• Masked emails (`is*****@domain`) and phones (`******4355`)<br>• Extracted birth year only | **1,000 unique passengers**<br>(P1000 to P1999) |
| **`payments`** | 48 null amounts<br>30 rows with text 'INVALID'<br>267 bookings had multiple payments | • Converted amount column to numeric (`errors='coerce'`)<br>• Flagged all 78 missing amounts with `is_amount_missing = True`<br>• Kept records for transaction counts without guessing fake money | **1,000 payments**<br>(922 valid amounts totaling ₹7.37M) |

---

## Privacy & PII Protection (DPDP Act Compliance)

Since passenger personal information was exposed in plain text, I added privacy protections:
* **Aadhaar ID & Passport Number:** Converted using **SHA-256** one-way cryptographic hashing (`hashlib.sha256`).
* **Email Address:** Masked to show only the first two characters (e.g. `is*****@rediffmail.com`).
* **Phone & Emergency Contacts:** Masked to show only the last four digits (e.g. `******4355`).
* **Date of Birth:** Stored only the birth year (e.g. `1984`) to allow demographic analysis while protecting full DOB.

---

## Key Findings from the 7 KPIs

1. **Average Flight Duration (KPI 1):** All four airlines have almost the exact same average duration (~163 to 165 minutes, or ~2.7 hours). Domestic flight times depend on the route distance, not the airline.
2. **Busiest Routes (KPI 2):** **BOM -> CCU** is the busiest route with **90 flights**, followed by **CCU -> DEL** with **72 flights**.
3. **Flight Delays & Anomalies (KPI 3):** Using a 2-sigma threshold per route ($	ext{Mean} + 2 	imes 	ext{Std}$), only **1 flight** (`6F212` by IndiGo on `BLR -> CCU`) was flagged as an anomaly. It took 282 minutes (~4.7 hours) when that route usually takes ~128 minutes.
4. **Airline Market Share (KPI 4):** IndiGo operates the most flights (**27.1%**), followed by Air India (**25.4%**), SpiceJet (**24.5%**), and Vistara (**22.9%**).
5. **Booking Status Breakdown (KPI 5):** **34.4%** of bookings are Cancelled (including invalid ones), **32.0%** are Confirmed, and **29.1%** are Pending.
6. **Total Revenue by Airline (KPI 6):** Total revenue from valid payments was **₹7,370,442.88 (~₹7.37M)**. **Vistara made the highest revenue (₹2.13M)** even though it had the fewest flights, because its average ticket price is higher (~₹9,184).
7. **Top Revenue Routes (KPI 7):** The routes that brought in the most money were **BOM -> CCU** (~₹7.00 Lakhs) and **CCU -> DEL** (~₹5.83 Lakhs), matching the high-traffic routes.

---

## Power BI Setup

To view the data in Power BI:
1. Load the 4 CSV files from `cleaned_data/`.
2. Set up the relationships in Model View:
   - `fact_bookings[flight_id]` $
ightarrow$ `dim_flights[flight_id]`
   - `fact_bookings[passenger_id]` $
ightarrow$ `dim_passengers[passenger_id]`
   - `dim_payments[booking_id]` $
ightarrow$ `fact_bookings[booking_id]`
3. Core DAX formulas used:
   - `Total Revenue = CALCULATE(SUM(dim_payments[amount]), dim_payments[is_amount_missing] = FALSE)`
   - `Total Flights = COUNTROWS(dim_flights)`
   - `Avg Duration = AVERAGE(dim_flights[duration_minutes])`
   - `Cancellation Rate = DIVIDE(CALCULATE(COUNTROWS(fact_bookings), fact_bookings[status] = "CANCELLED"), COUNTROWS(fact_bookings), 0)`

---

## Deliverables Checklist

- [x] Clean, well-documented Jupyter notebook: [`Data_Cleaning.ipynb`](Data_Cleaning.ipynb)
- [x] Cleaned datasets in CSV format: [`cleaned_data/`](cleaned_data/)
- [x] Detailed documentation report: [`ASG_Airlines_Data_Engineering_Documentation.docx`](ASG_Airlines_Data_Engineering_Documentation.docx)
- [x] High-res charts and architecture diagrams: [`visuals/`](visuals/)
- [x] GitHub README walkthrough: [`README.md`](README.md)
