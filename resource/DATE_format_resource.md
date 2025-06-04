# Comprehensive Date Handling Guide for Data Engineers

## Table of Contents
1. [Date Data Types Overview](#date-data-types-overview)
2. [Date Parsing and Conversion](#date-parsing-and-conversion)
3. [Date Transformation Patterns](#date-transformation-patterns)
4. [Date Arithmetic Operations](#date-arithmetic-operations)
5. [Time Zone Handling](#time-zone-handling)
6. [Date Validation and Cleaning](#date-validation-and-cleaning)
7. [Date Aggregations](#date-aggregations)
8. [Date Dimension Tables](#date-dimension-tables)
9. [Performance Considerations](#performance-considerations)
10. [Real-World Scenarios](#real-world-scenarios)

## Date Data Types Overview

### Common Date/Time Types Across Platforms

| Database | Date Type | Timestamp Type | Time Type | Interval Type |
|----------|----------|---------------|----------|--------------|
| PostgreSQL | `DATE` | `TIMESTAMP`/`TIMESTAMPTZ` | `TIME` | `INTERVAL` |
| MySQL | `DATE` | `DATETIME`/`TIMESTAMP` | `TIME` | - |
| SQL Server | `DATE` | `DATETIME`/`DATETIME2` | `TIME` | - |
| BigQuery | `DATE` | `TIMESTAMP`/`DATETIME` | `TIME` | - |
| Snowflake | `DATE` | `TIMESTAMP_*` | `TIME` | - |
| Spark | `DateType` | `TimestampType` | - | - |

### Example: Creating Date Columns

```sql
-- PostgreSQL example
CREATE TABLE events (
    event_id SERIAL PRIMARY KEY,
    event_date DATE,
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
    duration INTERVAL
);

-- Spark DataFrame
from pyspark.sql.types import DateType, TimestampType
from pyspark.sql.functions import current_date, current_timestamp

df = spark.createDataFrame([], schema="id INT")
df = df.withColumn("today", current_date()) \
       .withColumn("now", current_timestamp())
```

## Date Parsing and Conversion

### Common Parsing Scenarios

```python
# PySpark parsing examples
from pyspark.sql.functions import to_date, to_timestamp

df = df.withColumn("parsed_date", 
          to_date(col("date_string"), "MM/dd/yyyy")) \
       .withColumn("parsed_ts", 
          to_timestamp(col("ts_string"), "yyyy-MM-dd HH:mm:ss"))
```

### Handling Multiple Date Formats

```python
# Python function for handling messy dates
from datetime import datetime
from dateutil.parser import parse

def parse_flex_date(date_str):
    try:
        return parse(date_str, dayfirst=False)
    except:
        return None  # or handle differently

# Spark UDF application
from pyspark.sql.functions import udf
from pyspark.sql.types import TimestampType

parse_date_udf = udf(parse_flex_date, TimestampType())
df = df.withColumn("clean_date", parse_date_udf(col("raw_date")))
```

### Epoch Time Conversion

```sql
-- SQL conversions
-- PostgreSQL
SELECT to_timestamp(epoch_column) FROM table;

-- BigQuery
SELECT TIMESTAMP_SECONDS(epoch_int) FROM table;
SELECT TIMESTAMP_MILLIS(epoch_millis) FROM table;

-- Spark
from pyspark.sql.functions import from_unixtime
df = df.withColumn("datetime", from_unixtime(col("epoch_seconds")))
```

## Date Transformation Patterns

### Extracting Date Parts

```python
# PySpark date part extraction
from pyspark.sql.functions import year, month, dayofmonth, dayofweek

df = df.withColumn("year", year(col("date"))) \
       .withColumn("month", month(col("date"))) \
       .withColumn("day", dayofmonth(col("date"))) \
       .withColumn("dow", dayofweek(col("date")))
```

### Fiscal Year Calculations

```sql
-- Fiscal year starting April 1
SELECT 
    CASE 
        WHEN EXTRACT(MONTH FROM date_column) >= 4 
        THEN EXTRACT(YEAR FROM date_column) 
        ELSE EXTRACT(YEAR FROM date_column) - 1 
    END AS fiscal_year
FROM transactions;
```

### Date Formatting

```python
# Spark date formatting
from pyspark.sql.functions import date_format

df = df.withColumn("formatted_date", 
          date_format(col("date"), "yyyy-MM-dd")) \
       .withColumn("month_year", 
          date_format(col("date"), "MMMM yyyy"))
```

## Date Arithmetic Operations

### Date Differences

```sql
-- PostgreSQL
SELECT 
    event_date - created_date AS days_diff,
    AGE(event_date, created_date) AS precise_interval
FROM events;

-- Spark
from pyspark.sql.functions import datediff
df = df.withColumn("days_between", datediff(col("end_date"), col("start_date")))
```

### Date Addition/Subtraction

```python
# PySpark date math
from pyspark.sql.functions import date_add, date_sub, add_months

df = df.withColumn("next_week", date_add(col("date"), 7)) \
       .withColumn("prev_month", add_months(col("date"), -1))
```

### Workday Calculations

```python
# Python workday calculation (excluding weekends)
from datetime import timedelta
from dateutil import rrule

def add_workdays(start_date, days_to_add):
    return list(rrule.rrule(
        rrule.DAILY,
        dtstart=start_date,
        count=days_to_add+1,
        byweekday=[rrule.MO, rrule.TU, rrule.WE, rrule.TH, rrule.FR]
    ))[-1]
```

## Time Zone Handling

### Time Zone Conversion Patterns

```python
# PySpark time zone handling
from pyspark.sql.functions import from_utc_timestamp, to_utc_timestamp

df = df.withColumn("local_time", 
          from_utc_timestamp(col("utc_time"), "America/New_York")) \
       .withColumn("utc_time", 
          to_utc_timestamp(col("local_time"), "Asia/Tokyo"))
```

### Daylight Saving Time Considerations

```python
# Handling DST transitions
import pytz
from datetime import datetime

def convert_with_dst(dt, from_tz, to_tz):
    from_zone = pytz.timezone(from_tz)
    to_zone = pytz.timezone(to_tz)
    localized = from_zone.localize(dt)
    return localized.astimezone(to_zone)

# Example: Convert 2023-03-12 02:30 (when DST starts in US/Eastern)
convert_with_dst(
    datetime(2023, 3, 12, 2, 30),
    "US/Eastern",
    "UTC"
)
```

## Date Validation and Cleaning

### Common Date Quality Checks

```python
# PySpark date validation
from pyspark.sql.functions import col, to_date, lit, when
from pyspark.sql.types import BooleanType

# Validate date format
df = df.withColumn("is_valid_date", 
    to_date(col("date_str"), "yyyy-MM-dd").isNotNull())

# Identify future dates (often invalid)
df = df.withColumn("is_future_date", 
    col("date_col") > current_date())

# Weekend detection
df = df.withColumn("is_weekend", 
    dayofweek(col("date_col")).isin([1, 7]))
```

### Handling Invalid Dates

```python
# Date cleaning pipeline
from pyspark.sql.functions import coalesce, to_date, lit

clean_df = (df
    .withColumn("clean_date", 
        coalesce(
            to_date(col("date_str"), "yyyy-MM-dd"),
            to_date(col("date_str"), "MM/dd/yyyy"),
            to_date(col("date_str"), "dd-MMM-yyyy"),
            lit(None)  # Final fallback
        )
    )
    .filter(col("clean_date").isNotNull())  # Remove invalid dates
)
```

## Date Aggregations

### Time-Based Rollups

```sql
-- Daily, weekly, monthly aggregations
SELECT
    DATE_TRUNC('week', event_date) AS week_start,
    COUNT(*) AS event_count,
    SUM(value) AS total_value
FROM events
GROUP BY DATE_TRUNC('week', event_date)
ORDER BY week_start;
```

### Comparing Periods

```sql
-- Year-over-year comparison
SELECT
    EXTRACT(MONTH FROM sale_date) AS month,
    SUM(CASE WHEN EXTRACT(YEAR FROM sale_date) = 2022 THEN amount END) AS y2022,
    SUM(CASE WHEN EXTRACT(YEAR FROM sale_date) = 2023 THEN amount END) AS y2023,
    (SUM(CASE WHEN EXTRACT(YEAR FROM sale_date) = 2023 THEN amount END) - 
    SUM(CASE WHEN EXTRACT(YEAR FROM sale_date) = 2022 THEN amount END)) AS yoy_diff
FROM sales
GROUP BY EXTRACT(MONTH FROM sale_date)
ORDER BY month;
```

### Rolling Windows

```python
# Spark rolling 7-day average
from pyspark.sql.window import Window
from pyspark.sql.functions import avg

window_spec = Window.orderBy("date").rowsBetween(-6, 0)
df = df.withColumn("7_day_avg", avg(col("value")).over(window_spec))
```

## Date Dimension Tables

### Star Schema Implementation

```sql
-- Date dimension table
CREATE TABLE dim_date (
    date_id DATE PRIMARY KEY,
    day_of_week SMALLINT,
    day_name VARCHAR(10),
    month SMALLINT,
    month_name VARCHAR(10),
    quarter SMALLINT,
    year SMALLINT,
    is_weekend BOOLEAN,
    is_holiday BOOLEAN,
    fiscal_year SMALLINT,
    fiscal_quarter SMALLINT
);

-- Fact table with date foreign key
CREATE TABLE fact_sales (
    sale_id BIGINT,
    date_id DATE REFERENCES dim_date(date_id),
    amount DECIMAL(12,2)
);
```

### Generating Date Dimensions

```python
# Python date dimension generator
import pandas as pd
from datetime import datetime, timedelta

def generate_date_dim(start_date, end_date):
    dates = pd.date_range(start_date, end_date, freq='D')
    df = pd.DataFrame({'date_id': dates})
    df['day_of_week'] = df['date_id'].dt.dayofweek + 1
    df['day_name'] = df['date_id'].dt.day_name()
    df['month'] = df['date_id'].dt.month
    df['month_name'] = df['date_id'].dt.month_name()
    df['quarter'] = df['date_id'].dt.quarter
    df['year'] = df['date_id'].dt.year
    df['is_weekend'] = df['day_of_week'].isin([6,7])
    return df
```

## Performance Considerations

### Indexing Strategies

```sql
-- PostgreSQL index examples
CREATE INDEX idx_orders_date ON orders(order_date);
CREATE INDEX idx_events_year_month ON events(EXTRACT(YEAR FROM event_date), EXTRACT(MONTH FROM event_date));

-- For temporal range queries
CREATE INDEX idx_orders_date_range ON orders USING BRIN(order_date);
```

### Partitioning by Date

```python
# Spark date partitioning
df.write.partitionBy("year", "month", "day") \
    .parquet("/data/events/")
```

```sql
-- BigQuery partitioned table
CREATE TABLE sales.partitioned_sales (
    sale_id STRING,
    sale_datetime TIMESTAMP,
    amount FLOAT64
)
PARTITION BY DATE(sale_datetime);
```

## Real-World Scenarios

### Scenario 1: Sessionization

```python
# PySpark sessionization example
from pyspark.sql.window import Window
from pyspark.sql.functions import lag, sum as _sum, when

window_spec = Window.partitionBy("user_id").orderBy("event_time")

session_df = (df
    .withColumn("prev_time", lag("event_time").over(window_spec))
    .withColumn("new_session", 
        when(
            (col("event_time").cast("long") - col("prev_time").cast("long")) > 1800,  # 30 min threshold
            1
        ).otherwise(0))
    .withColumn("session_id", _sum("new_session").over(window_spec))
)
```

### Scenario 2: Cohort Analysis

```sql
-- Monthly retention cohort
WITH first_purchases AS (
    SELECT 
        user_id,
        DATE_TRUNC('month', MIN(purchase_date)) AS cohort_month
    FROM purchases
    GROUP BY user_id
)

SELECT
    fc.cohort_month,
    DATE_TRUNC('month', p.purchase_date) AS purchase_month,
    COUNT(DISTINCT p.user_id) AS users,
    COUNT(DISTINCT p.user_id) / MAX(COUNT(DISTINCT p.user_id)) OVER (PARTITION BY fc.cohort_month) AS retention_rate
FROM purchases p
JOIN first_purchases fc ON p.user_id = fc.user_id
GROUP BY fc.cohort_month, DATE_TRUNC('month', p.purchase_date)
ORDER BY fc.cohort_month, purchase_month;
```

### Scenario 3: Time-Based Feature Engineering

```python
# Feature engineering for ML
from pyspark.sql.functions import datediff

features_df = (df
    .withColumn("days_since_last_purchase", 
        datediff(current_date(), col("last_purchase_date")))
    .withColumn("purchase_time_of_day", 
        hour(col("purchase_timestamp")) * 60 + minute(col("purchase_timestamp")))
    .withColumn("is_weekend_purchase", 
        dayofweek(col("purchase_date")).isin([1,7]))
)
```

This comprehensive guide covers all critical aspects of date handling in data engineering pipelines, from basic parsing to advanced temporal analytics. The examples provided can be adapted to various database systems and big data processing frameworks.