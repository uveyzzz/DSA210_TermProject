# Instagram Activity and Physical Exercise Levels

## Motivation

Social media platforms such as Instagram have become an inseparable part of daily 
life, influencing not only communication patterns but also self-perception and 
motivation. Particularly, exposure to idealized body images and lifestyle content 
may alter individuals’ perceptions of their own bodies and influence exercise 
behavior.

This project aims to investigate whether daily Instagram activity — measured 
through time spent and interaction frequency — has a measurable relationship 
with physical activity habits recorded by Apple Watch. By combining digital 
behavior and physiological data, the project seeks to uncover behavioral patterns 
that may illustrate how social media usage correlates with real-world activity levels.

---

## Data Sources

### Instagram Activity Data

Exported from Meta Account Center. Includes:
- Daily time spent on Instagram (in minutes)
- Number of sessions
- General engagement metrics

### Apple Watch Fitness Data

Exported from the Fitness app, focusing on Traditional Strength Training sessions. 
Includes:
- Workout duration  
- Effort levels  
- Calories burned  
- Timestamps

## Integration

The two datasets will be **merged on a daily level** using the common `date` field 
(`YYYY-MM-DD` format). Each record will represent one day of combined Instagram and 
Apple Watch data.  

After preprocessing, the final merged dataset will include variables such as:

| Variable | Description |
|-----------|-------------|
| date | Calendar day |
| time_spent_minutes | Total minutes spent on Instagram |
| session_count | Number of Instagram sessions |
| total_interactions | Total engagement (likes + comments + story replies) |
| total_workout_minutes | Daily total workout duration (minutes) |
| calories_burned_total | Daily total calories burned |
| effort_level_avg | Average effort intensity |
| heart_rate_avg | Average heart rate (if available) |
| workout_done | 1 if exercise occurred that day, 0 otherwise |

Missing values will be handled through imputation (e.g., filling with 0 for missing 
workouts). The merge will primarily use an **inner join** to ensure both data sources 
have valid daily entries; however, outer joins may be tested for sensitivity analysis.

---

## Planned Analysis

1. **Data Cleaning & Merging:**  
   Filter and preprocess both datasets, ensuring consistent date formats and units.

2. **Exploratory Data Analysis (EDA):**  
   Visualize daily Instagram usage vs. exercise duration.  
   Plot time-series trends and compute pairwise correlations.

3. **Statistical Testing:**  
   Test the hypothesis that higher Instagram activity is associated with lower workout 
   probability.  
   Compare mean Instagram time between workout and non-workout days using a t-test 
   or Mann–Whitney test.

4. **Preliminary Modeling (Future Step):**  
   Apply simple regression or classification models to predict workout likelihood from 
   Instagram activity metrics.

---

## Expected Findings

The analysis may reveal a **negative correlation** between social media usage time and 
daily workout engagement, suggesting that heavier Instagram use could correspond 
to decreased motivation or available time for exercise.

Alternatively, some patterns might indicate a **positive association** — for example, 
fitness-related content motivating users to exercise more frequently.

The project’s main goal is not to establish causality but to identify consistent patterns 
and statistical relationships.

---

## Limitations & Future Work

- Single-person data (individual case study) limits generalization.  
- Possible missing days or inaccurate activity logs.  
- Instagram dataset lacks content classification; future versions may add sentiment or 
  topic analysis to categorize exposure to body-ideal imagery.  
- Extension ideas: include sleep duration or step count to broaden behavioral analysis.

---

## Ethical Statement

This project and its analysis were fully conceptualized, designed, and implemented by 
the author. AI-based tools (e.g., ChatGPT) were used **only to improve the clarity, 
structure, and language of the written text**. All core ideas, data collection, coding, 
and interpretation processes belong entirely to the author.

The use of AI tools followed academic integrity principles, ensuring that the final 
content reflects the author’s own understanding, reasoning, and analytical work.

> *This README represents the project proposal for the DSA 210 course, focusing on 
> the intersection of social media behavior and physical activity data analytics.*
