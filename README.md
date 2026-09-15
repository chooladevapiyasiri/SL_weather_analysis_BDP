# Harnessing Big Data for Climate Insights: A Decade of Sri Lankan Weather Trends

## Executive Summary

This project analyses **14 years (2010–June 2024) of daily weather observations across every district of Sri Lanka**, using a multi-framework big data architecture — **Hadoop MapReduce, Apache Hive, and Apache Spark** — to extract monthly, seasonal, and extreme-weather trends from a large historical meteorological dataset. Beyond descriptive analytics, the project extends into **predictive modelling with Spark MLlib**, forecasting low evapotranspiration (ET₀) events to support irrigation and water-resource planning, with an interactive **Tableau** dashboard presenting the results for decision-makers.

**Live Dashboard:** [View the interactive Tableau dashboard](https://public.tableau.com/views/SriLankaAnalyticsDashboard_17677487310780/Overview?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)

**Medium Article 1:** [Harnessing Big Data for Climate Insights: Decoding a Decade of Weather Trends Across Sri Lanka](https://medium.com/@chooladevapiyasiri/harnessing-big-data-for-climate-insights-decoding-a-decade-of-weather-trends-across-sri-lanka-5d74f8ca4fb2)

**Medium Article 2:** [Predicting Low Evapotranspiration Events in Sri Lanka Using Apache Spark MLlib](https://medium.com/@chooladevapiyasiri/predicting-low-evapotranspiration-events-in-sri-lanka-using-apache-spark-mllib-9b4c18400cb0)

## Project Objectives

- Track monthly and seasonal trends in precipitation and temperature across all districts
- Identify extreme weather events and hotspots, including periods of unusually high rainfall or heat
- Calculate seasonal evapotranspiration and radiation exposure for agricultural planning
- Predict low evapotranspiration (ET₀) events and identify the dominant weather conditions behind them
- Provide high-level, interactive insights for decision-making, assisting authorities and analysts in planning and mitigation

## Overview of the Data

The dataset is built from two primary sources, joined on a common `location_id`:

- **Historical Weather Observations (2010–June 2024)** — daily temperature, precipitation, wind speed, shortwave radiation, sunshine duration, and evapotranspiration.
- **Geographical & Location Data** — city names, district identifiers, latitudes, longitudes, and elevations, enabling precise mapping of weather metrics.

Data preparation involved standardising feature columns and date formats, linking location IDs to weather observations, and removing missing/invalid records — producing a robust daily weather record for every district in Sri Lanka over a 14.5-year period, suitable for large-scale distributed processing and modelling.

## Big Data Architecture & Descriptive Analysis

A multi-framework pipeline was used, with each tool matched to the type of processing required:

### Hadoop MapReduce — Aggregation & Extreme Event Detection
- **Monthly aggregation per district:** weather and location datasets were joined in a MapReduce job — the Mapper tagged and emitted records by location ID, and the Reducer aggregated total precipitation and computed average temperature per city, year, and month.
- **Wettest month detection:** a second job summed total precipitation across all districts per month and tracked the running maximum to identify the wettest month in the dataset.
- **Finding:** the month with the highest total precipitation across the dataset was **November 2021**.

### Apache Hive — Scalable SQL Querying
- External tables and lightweight views were created over the location and weather data to support scalable SQL-based analysis.
- **Top 10 most temperate cities:** identified via average maximum temperature per city.
- **Seasonal evapotranspiration:** average evapotranspiration was calculated per city and year across Sri Lanka's two major agricultural seasons — **Sep–Mar** and **Apr–Aug**.

### Apache Spark — Scalable In-Memory Analytics
- **Shortwave radiation analysis:** days with shortwave radiation above 15 MJ/m² were flagged, then aggregated by city, year, and month to compute the monthly percentage of high-radiation days.
- **Weekly maximum temperatures:** the top 3 hottest months of each year (by average max temperature) were identified using window functions, then weekly maximum temperatures within those months were computed to analyse extreme heat patterns.

## Predictive Modelling — Low Evapotranspiration Events (Spark MLlib)

Extending beyond descriptive analytics, a predictive model was built to anticipate **low evapotranspiration (ET₀) events** during May — a transitional, agriculturally sensitive month between monsoon phases.

- **Labelling approach:** low ET₀ events were defined data-drivenly as observations below the **25th percentile** of May ET₀ values, which also introduced class imbalance that was accounted for during evaluation.
- **Features:** precipitation hours, sunshine duration, and max wind speed — established physical drivers of evapotranspiration, used without needing explicit scaling for the tree-based models.
- **Models compared:** Linear Regression (baseline), Random Forest Regressor, and Gradient Boosted Trees, each tuned via grid search and evaluated with 5-fold cross-validation on RMSE, MAE, and R².
- **Best model — Random Forest:** RMSE ≈ 0.48, MAE ≈ 0.37, **R² ≈ 0.84**, with a **recall of ~93%** for correctly identifying low-ET₀ cases — important since missing such events risks over-irrigation or water misallocation.
- **Findings:** low ET₀ events in May are associated with moderate-to-high precipitation durations, reduced sunshine hours, and moderate wind speeds insufficient to offset moisture saturation — consistent with cloud-cover-dominated, pre-monsoon transition conditions. Feature importance confirmed precipitation and sunshine duration as the strongest drivers, and residual diagnostics (visualised in Apache Zeppelin) showed no significant model bias across districts.

## Tableau Dashboard

The results are presented across **three interactive dashboards**, each built on top of the processed outputs:

### Overview
A district-level map, weekly max temperature trend, extreme-events chart, and heatmap, alongside KPI cards for **% High-Radiation Months (>15 MJ/m²)**, **Average Evapotranspiration (Apr–Aug and Sep–Mar)**, **Extreme Events count**, and the **Most Precipitous Month**.

### Precipitation Analysis
Heatmaps, bar charts, and a trendline of precipitation by district and month, with KPIs for **Total Precipitation**, **% of Months Above Threshold**, and the **Highest/Lowest Precipitation District and Month** — plus a "Top 5 Districts by Highest Precipitation" ranking.

### Temperature Analysis
Bar charts and heatmaps of temperature by district and month, districts exceeding 30°C, and KPIs for the **Hottest Month, Hottest Region, Hottest Year**, and **% of Hot Months per Year**.

**Key calculated metrics driving the dashboards** include an extreme-day flag (precipitation ≥ 50mm **and** wind gusts ≥ 80km/h), hot-month counts per district and year (mean temperature > 30°C), district-level precipitation ranking, and wet-vs-dry seasonal percentage change.

## Tech Stack

- **Distributed Processing:** Hadoop MapReduce, Apache Hive, Apache Spark (PySpark, Spark MLlib)
- **Data Processing:** Python, SQL (HiveQL)
- **Machine Learning:** Linear Regression, Random Forest Regressor, Gradient Boosted Trees (Spark MLlib), cross-validated hyperparameter tuning
- **Visualization:** Tableau (calculated fields, LOD expressions, parameters, dashboards), Apache Zeppelin (model diagnostics)

## Conclusion

This project demonstrates how complementary big data frameworks can be orchestrated for climate analytics: **Hadoop MapReduce** for efficient large-scale aggregation of precipitation and temperature trends, **Hive** for scalable SQL-based seasonal and city-level querying, and **Spark** for both fast in-memory descriptive processing and predictive modelling of extreme conditions. Moving from descriptive reporting to a validated predictive model for low evapotranspiration events (R² ≈ 0.84, recall ≈ 93%) shows how the same data pipeline can support proactive irrigation and water-resource planning, not just retrospective analysis. Combined with an interactive Tableau front end, the result is a robust framework for turning over a decade of raw meteorological data into insights that support climate-sensitive planning across Sri Lanka.
