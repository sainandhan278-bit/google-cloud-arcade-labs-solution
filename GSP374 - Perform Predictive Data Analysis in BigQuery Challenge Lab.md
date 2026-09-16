# GSP374 - Perform Predictive Data Analysis in BigQuery: Challenge Lab

Google Cloud manual solution for the **GSP374: Perform Predictive Data Analysis in BigQuery: Challenge Lab** --- loading soccer event data into BigQuery, writing analytical queries, creating BigQuery ML user-defined functions, training a logistic regression expected goals model, and applying it to 2018 World Cup data.

> Replace the lab-specific values with the values shown in your own lab panel before following along. The key variable that changes every session is the **number suffix** on your events table (e.g. `events968`, `events401`). See the "About Your Lab Variables" section below for what to replace.

## Author

**Aayush Shrestha (Shinux)** - Computer Engineer

**Portfolio:** [aayushshrestha7.com.np](https://aayushshrestha7.com.np/) | **GitHub:** [aayush105](https://github.com/aayush105) | **LinkedIn:** [aayushrestha](https://www.linkedin.com/in/aayushrestha/)

[![Watch on YouTube](https://img.shields.io/badge/Watch_on_YouTube-FF0000?style=for-the-badge&logo=youtube&logoColor=white)](https://www.youtube.com/@Er.Shinux)

---

## Lab Overview

This challenge lab has five tasks:

1. **Task 1:** Load soccer event data from Cloud Storage into two BigQuery tables (JSON and CSV formats).
2. **Task 2:** Write a query that shows the penalty kick success rate for each player.
3. **Task 3:** Write a query to analyze shot distance and goal percentage from shot distance.
4. **Task 4:** Create two user-defined functions (shot distance and shot angle) and train a BigQuery ML logistic regression expected goals model.
5. **Task 5:** Use the trained model to make predictions on 2018 World Cup shot data.

> **Important:** This is a **manual solution** - you do not need to download or execute any GitHub script. Use the lab-provided values for your dataset name and events table name.

---

## ⚠️ About Your Lab Variables

Every lab session uses a **different number suffix** on the events table and related names. In this guide, `968` is used as an example. **Find your suffix from the events table name shown in your lab panel** and replace accordingly everywhere.

| Placeholder used in this guide | What to replace it with |
| ------------------------------- | ----------------------- |
| `events968` | Your events table name (e.g. `events401`) |
| `GetShotDistanceToGoal968` | Same suffix (e.g. `GetShotDistanceToGoal401`) |
| `GetShotAngleToGoal968` | Same suffix (e.g. `GetShotAngleToGoal401`) |
| `xg_logistic_reg_model_968` | Same suffix (e.g. `xg_logistic_reg_model_401`) |

> **Quick rule:** Whatever number comes after `events` in your lab (e.g. `401` in `events401`), use that same number everywhere you see `968` in this guide.

---

# TASK 1 - Data Ingestion

## Step 1 - Open BigQuery and Locate the Dataset

1. In the Google Cloud console, go to **Navigation menu → BigQuery**.
2. In the **Explorer** panel on the left, click your **Project ID** to expand it.
3. You will see a dataset called **`soccer`** — click it to expand.

---

## Step 2 - Load the JSON Events Table

1. Click the **three-dot menu (⋮)** next to the `soccer` dataset → **Create table**.
2. Fill in the following fields:

| Field | Value |
| ----- | ----- |
| Create table from | **Google Cloud Storage** |
| Select file from Cloud Storage bucket | `spls/bq-soccer-analytics/events.json` |
| File format | **JSONL (Newline delimited JSON)** |
| Dataset | `soccer` |
| Table name | **Your events table name from the lab panel** (e.g. `events968`) |
| Schema | Check **Auto detect** |

3. Click **Create table** and wait for the job to complete.

---

## Step 3 - Load the CSV Tags Table

1. Click the **three-dot menu (⋮)** next to the `soccer` dataset → **Create table** again.
2. Fill in the following fields:

| Field | Value |
| ----- | ----- |
| Create table from | **Google Cloud Storage** |
| Select file from Cloud Storage bucket | `spls/bq-soccer-analytics/tags2name.csv` |
| File format | **CSV** |
| Dataset | `soccer` |
| Table name | **Your tags table name from the lab panel** (e.g. `tags2name`) |
| Schema | Check **Auto detect** |

3. Click **Create table** and wait for the job to complete.

> ## Click "Check my progress" for Task 1 - Check tables are created

---

# TASK 2 - Analyze Soccer Data (Penalty Kick Success Rate)

## Step 4 - Run the Penalty Kick Query

Open a **new query tab** in BigQuery and run the following.

> **Replace `events968`** with your actual events table name from the lab panel.

```sql
SELECT
  PlayerId,
  (Players.firstName || ' ' || Players.lastName) AS playerName,
  COUNT(id) AS numPKAtt,
  SUM(IF(101 IN UNNEST(tags.id), 1, 0)) AS numPKGoals,
  SAFE_DIVIDE(
    SUM(IF(101 IN UNNEST(tags.id), 1, 0)),
    COUNT(id)
  ) AS PKSuccessRate
FROM `soccer.events968` Events
LEFT JOIN
  `soccer.players` Players
ON
  Events.playerId = Players.wyId
WHERE
  eventName = 'Free Kick'
  AND subEventName = 'Penalty'
GROUP BY
  playerId, playerName
HAVING
  numPKAtt >= 5
ORDER BY
  PKSuccessRate DESC, numPKAtt DESC
```

> **Note:** Tag `101` represents a goal. The query joins events with the players table, filters for penalty kicks, and only includes players with at least 5 attempts.

> ## Click "Check my progress" for Task 2 - Check penalty kick success rate

---

# TASK 3 - Gain Insight by Analyzing Shot Distance

## Step 5 - Run the Shot Distance Analysis Query

Open a **new query tab** in BigQuery and run the following.

> **Replace `events968`** with your actual events table name.

```sql
WITH
Shots AS
(
  SELECT
    *,
    (101 IN UNNEST(tags.id)) AS isGoal,
    SQRT(
      POW(
        ((100 - positions[ORDINAL(1)].x) * 90/60), 1
      ) +
      POW(
        ((60 - positions[ORDINAL(1)].y) * 118/83), 2
      )
    ) AS shotDistance
  FROM `soccer.events968`
  WHERE
    eventName = 'Shot' OR
    (eventName = 'Free Kick' AND subEventName
    IN ('Free kick shot', 'Penalty'))
)

SELECT
  ROUND(shotDistance, 0) AS ShotDistRound0,
  COUNT(*) AS numShots,
  SUM(IF(isGoal, 1, 0)) AS numGoals,
  AVG(IF(isGoal, 1, 0)) AS goalPct
FROM Shots
WHERE shotDistance <= 50
GROUP BY ShotDistRound0
ORDER BY ShotDistRound0
```

> **Note:** The midpoint of the goal mouth is `(100, 60)` in the 0–100 coordinate system. Field dimensions used: ~90m (x-axis) and ~118m/83 (y-axis conversion). Only shots within 50 metres are included.

> ## Click "Check my progress" for Task 3 - Analyze shot distance

---

# TASK 4 - Create a Regression Model Using Soccer Data

## Step 6 - Create the Shot Distance Function

The lab provides this code directly. Open a **new query tab**, copy the code block shown in **your lab panel for "Calculate shot distance"**, and run it.

> The function will be named something like `soccer.GetShotDistanceToGoal968` — use the exact name shown in your lab.

Click **Run** and wait for the function to be created successfully.

> ## Click "Check my progress" for Task 4 - Calculate shot distance

---

## Step 7 - Create the Shot Angle Function

The lab provides this code directly. Open a **new query tab**, copy the code block shown in **your lab panel for "Calculate shot angle"**, and run it.

> The function will be named something like `soccer.GetShotAngleToGoal968` — use the exact name shown in your lab.

Click **Run** and wait for the function to be created successfully.

> ## Click "Check my progress" for Task 4 - Calculate shot angle

---

## Step 8 - Create the Expected Goals Logistic Regression Model

Open a **new query tab** and run the following.

> **Replace `968`** with your suffix everywhere it appears (`events968`, `GetShotDistanceToGoal968`, `GetShotAngleToGoal968`, `xg_logistic_reg_model_968`).

```sql
CREATE MODEL `soccer.xg_logistic_reg_model_968`
OPTIONS(
  model_type = 'LOGISTIC_REG',
  input_label_cols = ['isGoal']
) AS

SELECT
  Events.subEventName AS shotType,
  (101 IN UNNEST(Events.tags.id)) AS isGoal,
  `soccer.GetShotDistanceToGoal968`(Events.positions[ORDINAL(1)].x,
    Events.positions[ORDINAL(1)].y) AS shotDistance,
  `soccer.GetShotAngleToGoal968`(Events.positions[ORDINAL(1)].x,
    Events.positions[ORDINAL(1)].y) AS shotAngle

FROM `soccer.events968` Events
LEFT JOIN `soccer.matches` Matches ON Events.matchId = Matches.wyId
LEFT JOIN `soccer.competitions` Competitions ON Matches.competitionId = Competitions.wyId

WHERE
  Competitions.name != 'World Cup'
  AND (
    eventName = 'Shot'
    OR (eventName = 'Free Kick' AND subEventName IN ('Free kick shot', 'Penalty'))
  );
```

> **Note:** World Cup matches are **excluded** from model training — they are saved for prediction in Task 5. This query may take **2–5 minutes** to complete as it trains the ML model.

After training completes, click **Go to model** in the Query results section, then navigate to the **EVALUATION** tab to review **Log loss** and **ROC AUC** metrics.

> ## Click "Check my progress" for Task 4 - Create BigQuery logistic regression model

---

# TASK 5 - Make Predictions from New Data with the BigQuery Model

## Step 9 - Run Predictions on 2018 World Cup Shots

Open a **new query tab** and run the following.

> **Replace `968`** with your suffix everywhere it appears (`events968`, `GetShotDistanceToGoal968`, `GetShotAngleToGoal968`, `xg_logistic_reg_model_968`).

```sql
SELECT
  predicted_isGoal_probs[ORDINAL(1)].prob AS predictedGoalProb,
  * EXCEPT (predicted_isGoal, predicted_isGoal_probs),
FROM
  ML.PREDICT(
    MODEL `soccer.xg_logistic_reg_model_968`,
    (
      SELECT
        Events.playerId,
        (Players.firstName || ' ' || Players.lastName) AS playerName,
        Teams.name AS teamName,
        CAST(Matches.dateutc AS DATE) AS matchDate,
        Matches.label AS match,
        CAST((CASE
          WHEN Events.matchPeriod = '1H' THEN 0
          WHEN Events.matchPeriod = '2H' THEN 45
          WHEN Events.matchPeriod = 'E1' THEN 90
          WHEN Events.matchPeriod = 'E2' THEN 105
          ELSE 120
        END) +
        CEILING(Events.eventSec / 60) AS INT64)
        AS matchMinute,
        Events.subEventName AS shotType,
        (101 IN UNNEST(Events.tags.id)) AS isGoal,
        `soccer.GetShotDistanceToGoal968`(Events.positions[ORDINAL(1)].x,
          Events.positions[ORDINAL(1)].y) AS shotDistance,
        `soccer.GetShotAngleToGoal968`(Events.positions[ORDINAL(1)].x,
          Events.positions[ORDINAL(1)].y) AS shotAngle
      FROM `soccer.events968` Events
      LEFT JOIN `soccer.matches` Matches ON Events.matchId = Matches.wyId
      LEFT JOIN `soccer.competitions` Competitions ON Matches.competitionId = Competitions.wyId
      LEFT JOIN `soccer.players` Players ON Events.playerId = Players.wyId
      LEFT JOIN `soccer.teams` Teams ON Events.teamId = Teams.wyId
      WHERE
        Competitions.name = 'World Cup'
        AND (
          eventName = 'Shot'
          OR (eventName = 'Free Kick' AND subEventName IN ('Free kick shot'))
        )
        AND (101 IN UNNEST(Events.tags.id))
    )
  );
```

> **Note:** This query filters to **only World Cup goals** (`101 IN UNNEST(Events.tags.id)`) and returns the model's predicted probability for each goal. A lower `predictedGoalProb` means the goal was harder to score — i.e., more impressive.

> ## Click "Check my progress" for Task 5 - Make predictions from the model

---

# Quick Reference — What to Replace

| What you see in this guide | Replace with |
| -------------------------- | ------------ |
| `events968` | Your events table name from lab panel |
| `GetShotDistanceToGoal968` | Same suffix as your events table |
| `GetShotAngleToGoal968` | Same suffix as your events table |
| `xg_logistic_reg_model_968` | Same suffix as your events table |

> The suffix (e.g. `968`) is the number after `events` in your events table name shown in the lab panel. It is the **only thing that changes** between lab sessions for the SQL queries in Tasks 2–5.

---

### ⚠️ Disclaimer

- **This guide and instructions are provided strictly for educational purposes to help you learn Google Cloud services and advance your engineering skills. Please review all commands thoroughly to understand the underlying infrastructure before execution. Always comply with Qwiklabs/Google Cloud Skills Boost Terms of Service and YouTube Community Guidelines. This material is designed to enhance hands-on learning, not to bypass lab challenges.**

### © Credit & Attribution

- **All educational content, lab scenarios, and original resources belong to [Google Cloud Skills Boost](https://www.cloudskillsboost.google/). No copyright infringement is intended. If you are a copyright owner and have concerns, please reach out via direct message for proper attribution or immediate content removal.** 🙏
