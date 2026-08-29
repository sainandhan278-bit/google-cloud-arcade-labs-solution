# Google Cloud COVID-19 Challenge Lab

BigQuery SQL solutions for the Google Cloud COVID-19 Challenge Lab (GSP787) using the `bigquery-public-data.covid19_open_data.covid19_open_data` dataset.

> Replace the placeholders (e.g. `<DATE>`, `<DEATH_COUNT>`) in each query with the values given by your own lab instance before running it.

## Author

**Aayush Shrestha (Shinux)** — Computer Engineer

**Portfolio:** [aayushshrestha7.com.np](https://aayushshrestha7.com.np/) | **GitHub:** [aayush105](https://github.com/aayush105) | **LinkedIn:** [aayushrestha](https://www.linkedin.com/in/aayushrestha/)

---

## Task 1. Total Confirmed Cases

```sql
SELECT
  SUM(cumulative_confirmed) AS total_cases_worldwide
FROM
  `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE
  date = '<DATE>';
```

## Task 2. Worst Affected Areas

```sql
SELECT
  COUNT(*) AS count_of_states
FROM (
  SELECT
    subregion1_name,
    SUM(cumulative_deceased) AS total_deaths
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name = 'United States of America'
    AND date = '<DATE>'
    AND subregion1_name IS NOT NULL
  GROUP BY
    subregion1_name
  HAVING
    SUM(cumulative_deceased) > <DEATH_COUNT>
);
```

## Task 3. Identify Hotspots

```sql
SELECT
  subregion1_name AS state,
  SUM(cumulative_confirmed) AS total_confirmed_cases
FROM
  `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE
  country_name = 'United States of America'
  AND date = '<DATE>'
  AND subregion1_name IS NOT NULL
GROUP BY
  subregion1_name
HAVING
  total_confirmed_cases > <CONFIRMED_CASES>
ORDER BY
  total_confirmed_cases DESC;
```

## Task 4. Fatality Ratio

```sql
SELECT
  SUM(cumulative_confirmed) AS total_confirmed_cases,
  SUM(cumulative_deceased) AS total_deaths,
  (SUM(cumulative_deceased) / SUM(cumulative_confirmed)) * 100 AS case_fatality_ratio
FROM
  `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE
  country_name = 'Italy'
  AND date BETWEEN '<START_DATE>' AND '<END_DATE>';
```

## Task 5. Identify a Specific Day

```sql
SELECT
  date
FROM (
  SELECT
    date,
    SUM(cumulative_deceased) AS total_deaths
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name = 'Italy'
  GROUP BY
    date
)
WHERE
  total_deaths > <DEATH_COUNT>
ORDER BY
  date ASC
LIMIT 1;
```

## Task 6. Find Days with Zero Net New Cases

```sql
WITH india_cases_by_date AS (
  SELECT
    date,
    SUM(cumulative_confirmed) AS cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name = 'India'
    AND date BETWEEN '<START_DATE>' AND '<END_DATE>'
  GROUP BY
    date
  ORDER BY
    date ASC
),

india_previous_day_comparison AS (
  SELECT
    date,
    cases,
    LAG(cases) OVER (ORDER BY date) AS previous_day,
    cases - LAG(cases) OVER (ORDER BY date) AS net_new_cases
  FROM
    india_cases_by_date
)

SELECT
  COUNT(*) AS days_with_zero_net_new_cases
FROM
  india_previous_day_comparison
WHERE
  net_new_cases = 0;
```

---
## Task 7. Doubling Rate

```sql
WITH us_cases_by_date AS (
  SELECT
    date,
    SUM(cumulative_confirmed) AS cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name = 'United States of America'
    AND date BETWEEN '<START_DATE>' AND '<END_DATE>'
  GROUP BY
    date
),

us_previous_day_comparison AS (
  SELECT
    date,
    cases,
    LAG(cases) OVER (ORDER BY date) AS previous_day
  FROM
    us_cases_by_date
)

SELECT
  date AS Date,
  cases AS Confirmed_Cases_On_Day,
  previous_day AS Confirmed_Cases_Previous_Day,
  ((cases - previous_day) / previous_day) * 100 AS Percentage_Increase_In_Cases
FROM
  us_previous_day_comparison
WHERE
  ((cases - previous_day) / previous_day) * 100 > <LIMIT_VALUE>
ORDER BY
  date;
```

## Task 8. Recovery Rate

```sql
WITH country_data AS (
  SELECT
    country_name AS country,
    SUM(cumulative_recovered) AS recovered_cases,
    SUM(cumulative_confirmed) AS confirmed_cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    date = '<DATE>'
  GROUP BY
    country_name
)

SELECT
  country,
  recovered_cases,
  confirmed_cases,
  (recovered_cases / confirmed_cases) * 100 AS recovery_rate
FROM
  country_data
WHERE
  confirmed_cases > 50000
ORDER BY
  recovery_rate DESC
LIMIT <LIMIT_VALUE>;
```

## Task 9. CDGR - Cumulative Daily Growth Rate

CDGR = `((last_day_cases / first_day_cases) ^ (1 / days_diff)) - 1`, calculated for France from the first reported case (Jan 24, 2020) to a given end date.

```sql
WITH france_cases AS (
  SELECT
    date,
    SUM(cumulative_confirmed) AS total_cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name = 'France'
    AND date IN ('2020-01-24', '<END_DATE>')
  GROUP BY
    date
  ORDER BY
    date
),

summary AS (
  SELECT
    total_cases AS first_day_cases,
    LEAD(total_cases) OVER (ORDER BY date) AS last_day_cases,
    DATE_DIFF(LEAD(date) OVER (ORDER BY date), date, DAY) AS days_diff
  FROM
    france_cases
  LIMIT 1
)

SELECT
  first_day_cases,
  last_day_cases,
  days_diff,
  POW((last_day_cases / first_day_cases), (1 / days_diff)) - 1 AS cdgr
FROM
  summary;
```

## Task 10. Create a Looker Studio Report

Chart the following for the United States using the query below as the custom BigQuery data source: **Confirmed Cases**, **Deaths**, filtered by **Date Range**.

```sql
SELECT
  date,
  SUM(cumulative_confirmed) AS country_cases,
  SUM(cumulative_deceased) AS country_deaths
FROM
  `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE
  country_name = 'United States of America'
  AND date BETWEEN '<START_DATE>' AND '<END_DATE>'
GROUP BY
  date
ORDER BY
  date;
```

---

## Author

**Aayush Shrestha (Shinux)** — Computer Engineer

**Portfolio:** [aayushshrestha7.com.np](https://aayushshrestha7.com.np/) | **GitHub:** [aayush105](https://github.com/aayush105) | **LinkedIn:** [aayushrestha](https://www.linkedin.com/in/aayushrestha/)