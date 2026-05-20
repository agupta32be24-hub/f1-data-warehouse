[F1_DBMS_README.md](https://github.com/user-attachments/files/28039886/F1_DBMS_README.md)
# 🏎️ F1 Data Warehouse & Analytics Platform

> A full-stack data engineering project implementing a **Medallion Architecture** (Bronze → Silver → Gold) on Microsoft SQL Server for Formula 1 historical data (2009–2024), with a **Flask web dashboard** for interactive analytics.

---

## 📌 Overview

Formula 1 generates millions of data points per race weekend — lap times, pit stop durations, qualifying positions, championship standings. This project transforms 14 raw F1 CSV files into a structured, query-optimised data warehouse using industry-standard Medallion Architecture, then surfaces the data through a Python Flask web application.

**Scope:** 2009–2024 | 1,125 races | 861 drivers | 77 circuits | 700,000+ lap-time records

---

## 🏗️ Architecture

```
CSV Files (14 sources, 700K+ records)
          │
          ▼
    ┌─────────────┐
    │ BRONZE LAYER│  Raw ingestion — all columns as NVARCHAR(500)
    │ 14 tables   │  BULK INSERT via stored procedure (timed + logged)
    └─────────────┘
          │
          ▼
    ┌─────────────┐
    │ SILVER LAYER│  Star Schema — typed, normalised, null-handled
    │ 4 dim + 4   │  dim_drivers, dim_circuits, dim_races, dim_constructors
    │ fact tables │  fct_race_results, fct_qualifying, fct_pit_stops, fct_lap_times
    └─────────────┘
          │
          ▼
    ┌─────────────┐
    │  GOLD LAYER │  Pre-aggregated KPI tables for dashboard
    │ 3 tables    │  driver_season_stats, constructor_season_stats, circuit_stats
    └─────────────┘
          │
          ▼
    ┌─────────────┐
    │  FLASK APP  │  4 routes — Home, Driver Profile, Standings, Head-to-Head
    │  pyodbc     │  Dynamic SQL-backed queries via Jinja2 templates
    └─────────────┘
```

---

## 📊 Dataset

| CSV File | Description | Records |
|---|---|---|
| circuits_clean.csv | F1 circuits worldwide | 77 |
| constructors.csv | Constructor teams | 212 |
| drivers_clean.csv | Driver biographic details | 861 |
| races.csv | Race events per season | 1,125 |
| results.csv | Race finishing results | 26,759 |
| lap_times.csv | Individual lap timings | 589,081 |
| qualifying.csv | Qualifying session results | 10,494 |
| pit_stops.csv | Pit stop durations | 11,371 |
| driver_standings.csv | Championship standings after each race | 34,863 |
| constructor_standings.csv | Constructor championship standings | 13,391 |
| sprint_results.csv | Sprint race results | 360 |

> **Source:** Ergast Formula 1 World Championship Historical Dataset (2009–2024)

---

## 🗄️ Database Design

### Silver Layer — Star Schema

| Table | Type | Key Columns |
|---|---|---|
| `silver.dim_drivers` | Dimension | driver_id, full_name, nationality, dob |
| `silver.dim_circuits` | Dimension | circuit_id, circuit_name, city, country, lat, lng |
| `silver.dim_races` | Dimension | race_id, season_year, round_number, circuit_id |
| `silver.dim_constructors` | Dimension | constructor_id, constructor_name, nationality |
| `silver.fct_race_results` | Fact | result_id, race_id, driver_id, points, position, laps |
| `silver.fct_qualifying` | Fact | qualify_id, race_id, driver_id, q1, q2, q3 |
| `silver.fct_pit_stops` | Fact | race_id, driver_id, stop_number, pit_duration_sec |
| `silver.fct_lap_times` | Fact | race_id, driver_id, lap, lap_time_ms |

### Gold Layer — Pre-aggregated Analytics

| Table | Purpose |
|---|---|
| `gold.driver_season_stats` | Wins, podiums, poles, DNFs, points, avg finish per driver per season |
| `gold.constructor_season_stats` | Team-level season KPIs and championship position |
| `gold.circuit_stats` | Circuit-level performance metrics |

---

## 🌐 Flask Web Application

Four interactive routes backed by live SQL queries:

| Route | Description | Data Source |
|---|---|---|
| `/` | Home — season selector and overview | `gold.driver_season_stats` |
| `/driver/<id>` | Driver profile — career stats, wins, podiums, season history | `gold.driver_season_stats`, `silver.fct_race_results` |
| `/standings/<year>` | Championship standings for a given season | `gold.driver_season_stats`, `gold.constructor_season_stats` |
| `/compare` | Head-to-head driver or constructor comparison across seasons | `gold.driver_season_stats` |

**Run locally:**
```bash
pip install flask pyodbc pandas
python app.py
```
> Requires Microsoft SQL Server with Windows Authentication (Trusted_Connection=yes).

---

## ⚙️ ETL Highlights

- **Bronze load:** `bronze.load_bronze` stored procedure using `BULK INSERT` with `FIRSTROW=2`, execution timing via `GETDATE()`, and per-table row-count validation
- **Silver transform:** `INSERT...SELECT` from Bronze with explicit type casting, `NULLIF()` for `\N` handling, and surrogate key assignment
- **Gold aggregation:** CTEs and `ROW_NUMBER() OVER (PARTITION BY ...)` window functions for deriving season-level KPIs and resolving mid-season team changes
- **Normalisation:** Silver dimension tables normalised to **3NF/BCNF**

---

## 🛠️ Tech Stack

![SQL Server](https://img.shields.io/badge/SQL%20Server-CC2927?style=flat&logo=microsoftsqlserver&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/Flask-000000?style=flat&logo=flask&logoColor=white)

- **Database:** Microsoft SQL Server, T-SQL, SSMS
- **ETL:** Stored Procedures, BULK INSERT, CTEs, Window Functions
- **Web App:** Python Flask, pyodbc, Jinja2
- **Data:** Pandas, CSV

---

## 📁 Repository Structure

```
f1-data-warehouse/
│
├── scripts/
│   ├── init_database.sql          # Creates F1_DW database + schemas
│   ├── bronze/
│   │   ├── DDL_bronze.sql         # Bronze layer table definitions
│   │   └── Proc_load_bronze.sql   # BULK INSERT stored procedure
│   ├── silver/
│   │   ├── DDL_Silver.sql         # Silver layer star-schema DDL
│   │   └── Insert_table_silver.sql # Bronze → Silver transformations
│   └── gold/
│       ├── DDL_gold.sql           # Gold layer aggregation tables
│       └── gold_insert_data.sql   # Silver → Gold aggregations
│
├── dataset/                       # 14 source CSV files
├── project overview.jpeg          # Architecture diagram
├── report.docx                    # Full project report
└── README.md
```

---

## 👥 Team

Submitted for **UCS613 – Database Management Systems Lab**, Even Semester 2025–26, Thapar Institute of Engineering & Technology.

| S.No. | Member | Roll No. | Group |
|---|---|---|---|
| 1 | Punya Chhabra | 1024030684 | 2C32 |
| 2 | Arnav Gupta | 1024030780 | 2C32 |
| 3 | Aditya Kumar | 1024030452 | 2C32 |

**Submitted To:** Ms. Abhishelly

---

## 📄 References

- [Ergast F1 Developer API](http://ergast.com/mrd/)
- [Microsoft SQL Server Documentation](https://docs.microsoft.com/en-us/sql/sql-server/)
- [Flask Documentation](https://flask.palletsprojects.com/)
