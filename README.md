
# 🚕 azure-nyc-taxi-pipeline

A modern data pipeline to analyze NYC Taxi data using Azure Data Factory, Azure Data Lake Storage, Synapse Analytics, and Power BI. The project ingests raw trip data, cleans and transforms it using PySpark notebooks on Azure Synapse, and visualizes trends like trip duration, fare amount, and pickup/dropoff zones.

---

## 🛠️ Project Overview

This project demonstrates a scalable data pipeline for transportation analytics. The NYC Taxi dataset is processed via a medallion architecture using PySpark notebooks hosted in Azure Synapse. It helps track trends in urban transportation and pricing models using interactive Power BI dashboards.

---

## 🧰 Tech Stack

- **Azure Data Factory** – Pipeline orchestration and data ingestion
- **Azure Data Lake Storage Gen2** – Centralized data storage
- **Azure Synapse Analytics** – Notebook-based transformations with PySpark
- **PySpark** – Data cleaning, transformation, and aggregation
- **Power BI** – Rich dashboards for visualization
- **NYC Taxi Dataset** – Public transportation data

---

## 🏗️ Architecture

```mermaid
graph TD;
  Source[NYC Taxi Dataset (CSV/Parquet)] --> ADF[Azure Data Factory]
  ADF --> Bronze[Bronze Layer - Raw Data]
  Bronze --> Silver[Silver Layer - Cleaned Data]
  Silver --> Gold[Gold Layer - Aggregated Data]
  Gold --> PowerBI[Power BI Dashboard]
```

---

## 📁 Notebooks Overview

### ⚪ Silver Layer (`silver_notebook_tan.ipynb`)

```python
# Load from Bronze
df_bronze = spark.read.format("delta").load("abfss://bronze@datalake/nyc_taxi")

# Basic cleaning
df_silver = df_bronze.filter("trip_distance > 0 and fare_amount > 0")

# Selecting necessary columns
df_silver = df_silver.select("pickup_datetime", "dropoff_datetime", "passenger_count", 
                             "trip_distance", "fare_amount", "pickup_location", "dropoff_location")

# Save as Silver
df_silver.write.format("delta").mode("overwrite").save("abfss://silver@datalake/nyc_taxi_cleaned")
```

### 🟡 Gold Layer (`gold_notebook.ipynb`)

```python
# Load Silver
df_silver = spark.read.format("delta").load("abfss://silver@datalake/nyc_taxi_cleaned")

# Aggregation for Power BI
df_gold = df_silver.groupBy("pickup_location").agg(
    avg("trip_distance").alias("avg_trip_distance"),
    avg("fare_amount").alias("avg_fare"),
    count("*").alias("total_trips")
)

# Save as Gold
df_gold.write.format("delta").mode("overwrite").save("abfss://gold@datalake/nyc_taxi_summary")
```

---

## 📊 Power BI Dashboard

- Import from Delta table in Data Lake
- Create visuals:
  - Average fare by pickup zone
  - Number of trips by time of day
  - Trip distance heatmaps

---

## ✅ Setup Steps

### 1. Create a Data Factory Pipeline

- Ingest data from NYC Open Data or Azure Blob to Bronze layer

### 2. Upload and Run Notebooks in Synapse

- Run `silver_notebook_tan.ipynb` to clean and transform the data
- Run `gold_notebook.ipynb` to aggregate for reporting

### 3. Connect Power BI

- Use Azure Data Lake as the source
- Load Gold table and create interactive visuals

---

## 💡 Notes

- Spark pools in Synapse should be configured for optimal performance.
- Use notebook scheduling for daily/weekly refresh.
- Make sure proper IAM roles are granted to access data lake and run pipelines.

