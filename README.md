# Instagram Activity and Physical Exercise Levels

## Motivation

Social media platforms such as Instagram have become an inseparable part of daily 
life, influencing not only communication patterns but also self-perception and 
motivation. In particular, repeated exposure to fitness content and idealized body 
images may shape how people think about exercise and “healthy lifestyles”.

This project investigates whether **my own Instagram fitness-related activity** 
(primarily liked posts) has a measurable relationship with my **actual workout 
behavior** recorded by Apple Watch. By combining digital behavior logs and fitness 
tracker data on a **daily** level, the project aims to uncover behavioral patterns 
between social media usage and real-world physical activity.

---

## Data Sources

### Instagram Activity Data

I exported my Instagram data from the **Meta Account Center** and focused on:

- `liked_posts.json` (and optionally `saved_posts.json`),
- `data_manual/saved_posts.json` (optional, used in the pipeline)


For each liked (or saved) post, I extracted:

- A timestamp (UNIX time)  
- The account handle (e.g. `title` field in the export)  
- Caption-like text if available (from `string_list_data` / `string_map_data`)

Using a simple keyword and handle-based heuristic, I classified each interaction as:

- **Fitness-related** (e.g., contains “gym”, “workout”, “fitness”, “strength”, 
  “sağlıklı yaşam”, or comes from a fitness-looking handle), or  
- **Non-fitness-related**

All interactions were then **aggregated to the daily level**, yielding variables such as:

- `total_like` – number of liked posts per day  
- `total_save` – number of saved posts per day (if used)  
- `fitness_like` – number of liked posts classified as fitness-related  
- `fitness_save` – number of saved posts classified as fitness-related  
- `fitness_total` – total fitness-related interactions (likes + saves)  
- `total_like_save` – total Instagram interactions (likes + saves)  
- `fitness_share_likesaves` – share of fitness-related interactions among all interactions  

### Apple Watch Fitness Data

I exported my workout history from the **Apple Fitness app** (via HealthAutoExport) 
as a CSV file (`Workouts-...csv`). Each row corresponds to a workout session.

From this file I used, for each workout:

- `Start` – start timestamp  
- `Duration` – in `HH:MM:SS` format  
- `Active Energy (kcal)`  
- `Resting Energy (kcal)`  
- `Avg. Heart Rate (count/min)`  
- `Max. Heart Rate (count/min)`  
- `Intensity (kcal/hr·kg)`  
- `Step Count`

These were aggregated **per calendar day** into:

- `total_workout_minutes` – sum of workout durations (in minutes)  
- `active_energy_total` – total active energy (kcal)  
- `resting_energy_total` – total resting energy (kcal)  
- `avg_heart_rate` – average of daily average heart rates  
- `max_heart_rate` – daily maximum heart rate  
- `intensity_avg` – average intensity  
- `step_count_total` – total daily steps  
- `workout_done` – binary flag: 1 if `total_workout_minutes > 0`, else 0  

---

## Integration and Feature Engineering

The two sources were merged into a **daily panel dataset**.

1. I first determined the overall calendar range covered by either source (from the 
   earliest date in Instagram data or workouts, to the latest one).
2. I generated a **full calendar** over this range (one row per day).
3. I left-joined:
   - the Instagram daily metrics (`df_fitness_daily`), and  
   - the workout daily metrics (`df_workouts_daily`)  
   onto this calendar.

Missing numeric values were filled with `0` (e.g., days with no workout or no likes).  
The resulting merged dataset has one row per day and includes, among others:

| Variable | Description |
|-----------|-------------|
| `date` | Calendar day (YYYY-MM-DD) |
| `total_like` | Number of liked posts that day |
| `total_save` | Number of saved posts that day (if any) |
| `fitness_like` | Number of liked fitness-related posts |
| `fitness_save` | Number of saved fitness-related posts |
| `fitness_total` | Total fitness-related IG interactions (likes + saves) |
| `total_like_save` | Total IG interactions (likes + saves) |
| `fitness_share_likesaves` | Fraction of interactions that are fitness-related |
| `total_workout_minutes` | Total workout duration (minutes) |
| `active_energy_total` | Total active energy (kcal) |
| `resting_energy_total` | Total resting energy (kcal) |
| `avg_heart_rate` | Average heart rate across workouts that day |
| `max_heart_rate` | Maximum heart rate across workouts that day |
| `step_count_total` | Total step count (from workout export) |
| `workout_done` | 1 if there was any workout that day, 0 otherwise |

For some analyses, I also created a **binned version** of daily fitness IG activity:

- `fitness_bin = 0` if `fitness_total = 0`  
- `fitness_bin = 1–2` if `1 ≤ fitness_total ≤ 2`  
- `fitness_bin = 3+` if `fitness_total ≥ 3`  

This binning is used to compare the probability of working out across different levels 
of Instagram fitness engagement.

---

## Analysis Performed (as of 30 November)

### 1. Data Cleaning & Merging

- Parsed Instagram timestamps (seconds or milliseconds) into timezone-aware datetimes, 
  then into plain dates.
- Built a fitness-content classifier based on:
  - English + Turkish keywords (e.g., “gym”, “workout”, “strength training”, 
    “sporsalonu”, “sağlıklı beslenme”),  
  - fitness-related hashtags, and  
  - typical patterns in account handles (e.g., containing “fitness”, “gym”, “coach”).
- Converted workout durations from `HH:MM:SS` strings into minutes.
- Aggregated all workouts per day and merged them with daily Instagram interaction counts.

The final merged dataset covers **370 days**, with:

- **140** workout days (`workout_done = 1`)  
- **230** rest days (`workout_done = 0`)

### 2. Exploratory Data Analysis (EDA)

I performed basic EDA, including:

- Histograms of `total_like_save` and `total_workout_minutes`, showing that both 
  variables are highly skewed (many low-activity days and a few very high-activity days).
- Time-series plots of daily workout duration and Instagram interactions over time.
- Scatter plots of:
  - `total_workout_minutes` vs `active_energy_total`  
  - `total_workout_minutes` vs `step_count_total`  

These confirmed **strong positive correlations** between workout duration and both 
active energy and step count, which serves as a sanity check that the Apple Health 
export is internally consistent.

### 3. Hypothesis Tests

#### RQ1 – Does fitness-related Instagram activity reduce my workout behavior?

**Initial hypothesis:**  
> On days when I interact more with fitness-related content on Instagram, I will be 
> **less likely** to work out (a “doom-scrolling instead of doing” effect).

To test this, I compared days with **zero** vs **at least one** fitness-related IG 
interaction (`fitness_total`):

- Days with `fitness_total = 0`:
  - Average workout duration ≈ **29.9 minutes**
  - Probability of working out (`P(workout_done = 1)`) ≈ **36.9%**
- Days with `1–2` fitness-related interactions:
  - Average workout duration ≈ **33.9 minutes**
  - Workout probability ≈ **40.0%**
- Days with `3+` fitness-related interactions:
  - Highest workout probability (≈ **60%**), but this bin contains only a few days.

A Welch two-sample t-test comparing **0 vs ≥1** fitness interactions gave:

- p-value ≈ **0.50**

This indicates **no statistically significant difference** in mean workout duration 
between days with and without fitness-related IG interactions. Non-parametric tests 
(Mann–Whitney U) led to the same qualitative conclusion.

In other words, my data do **not** support a strong negative relationship between 
fitness-related Instagram activity and working out. If anything, the averages suggest 
a weak **positive** trend (slightly more workout on days with some fitness IG activity), 
but the effect is small and very noisy.

#### RQ2 – Does overall Instagram interaction level relate to workouts?

I also compared daily **total IG interactions** (`total_like_save`) between workout and rest days:

- Average `total_like_save` was slightly higher on workout days than on rest days.
- A Welch t-test showed a p-value around **0.08** (not conventionally significant at 5%),
  while a Mann–Whitney U test indicated a small but significant shift in distributions 
  (p ≈ 0.02).

This suggests that on my own data, days with workouts may be associated with **slightly 
higher overall Instagram activity**, but this effect is also small and should be 
interpreted cautiously.

---
### 4. Machine Learning Models

For the 2 January milestone I added simple ML experiments in  
`02_machine_learning.ipynb` / `02_machine_learning.py`.

#### 4.1 Classification – Predicting Workout vs No-Workout

**Target:** `workout_done` (0/1)  
**Features:** daily Instagram metrics (`total_like_save`, `fitness_total`, 
`fitness_share_likesaves`), simple calendar features (day of week, month), and some
history features (e.g. “worked out yesterday”).

Models compared:

- Logistic Regression  
- Random Forest Classifier  

The main ROC curves are saved as:

- `figures/classification_results.png`

On the held-out test set:

- Accuracy is around the majority baseline (≈ 0.75–0.80).  
- ROC-AUC is ~0.54–0.57 (only slightly above random 0.5).  

So even with simple ML, Instagram + calendar features provide **very limited predictive 
power** for whether I will work out on a given day.

#### 4.2 Regression – Predicting Workout Minutes

For days with `workout_done = 1`, I tried to predict `total_workout_minutes`.

Models:

- Ridge Regression (linear)  
- Random Forest Regressor  

The “actual vs predicted” plot is saved as:

- `figures/regression_results.png`

R² scores are close to **0** (sometimes slightly positive for Random Forest, slightly 
negative for Ridge), and prediction errors are large. This means the models hardly 
explain any variance in workout duration beyond the global average.

> **ML conclusion:** My Instagram metrics and simple calendar/history features are **not
> sufficient to accurately predict** either whether I will work out or how long I will
> work out. The patterns are weak and noisy.

---


## Summary of Findings

- There is **no strong or statistically significant evidence** that higher 
  fitness-related Instagram activity makes me less likely to work out.  
- Basic descriptive statistics and hypothesis tests suggest that:
  - Workout duration and probability are **not lower** on “fitness IG days”; if anything, 
    they are slightly higher.  
  - Overall Instagram interactions (`total_like_save`) also show, at most, a weak 
    positive association with workout days.  
- Sanity-check correlations (workout minutes vs active energy / steps) are strongly 
  positive, supporting the internal validity of the Apple Health data.
- Machine-learning models (classification and regression) trained on Instagram + 
  calendar features perform only slightly better than chance and do **not** yield 
  useful prediction performance.

Overall, my personal dataset does **not confirm** the intuitive “doom-scrolling kills 
motivation” hypothesis. Any real effect—if it exists—appears to be **small and noisy** 
for this one-year period and one individual.

---
## HTML Summary Report

Please see the following website for the polished version of the whole project:
https://dsa-210-term-project-lttz.vercel.app

To present the results in a more polished way, I also created a small **HTML report**:
- Purpose: provide a clean, “dashboard-like” overview of the project outcome.

The HTML page includes:

- A short textual summary of the research question and dataset.  
- Embedded versions of the main figures:
  - distributions of Instagram interactions and workout duration,  
  - the bar chart of average workout minutes by fitness-interaction bins,  
  - the ROC curves for the workout classifier,  
  - the actual vs predicted minutes plot for the regression model.  
- Simple color coding and layout to mimic a professional analytics report.

---

## Limitations & Future Work

- The analysis is based on **a single person’s data**, so results are not generalizable.  
- Instagram data only covers **likes (and optionally saves)**, not full feed/story 
  impressions or watch time, which might be more relevant for “scrolling” behavior.  
- Fitness-related classification relies on simple keyword and handle rules and may 
  misclassify some posts.  
- Apple Health export may not capture all forms of physical activity (e.g., unlogged 
  walks or sports sessions).

Possible extensions:

- Improve content classification (e.g., include bio information, more refined 
  keyword lists, or even manual labeling of a subset).  
- Incorporate additional variables such as sleep duration or non-workout step counts.  
- Build simple predictive models (e.g., logistic regression) to predict `workout_done` 
  from daily Instagram features, and compare model performance to a baseline.  
- Replicate the analysis for multiple people to see if patterns are consistent across users.

---
## Ethical Statement

This project and its analysis were fully conceptualized, designed, and implemented by 
the author. AI-based tools (ClaudeAI, ChatGPT) were used **only to improve the clarity, 
structure, and language of the written text**. All core ideas, data collection, coding, 
and interpretation processes belong entirely to me.

The use of AI tools followed academic integrity principles, ensuring that the final 
content reflects the author’s own understanding, reasoning, and analytical work.

Note: I have used GitHub codespace and Claude.ai Opus 4.5 to develop the HTML website since it is out of scope for this course

> *This README represents the project proposal for the DSA 210 course, focusing on 
> the intersection of social media behavior and physical activity data analytics.*
