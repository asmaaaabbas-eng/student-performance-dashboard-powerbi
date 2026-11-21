# Student Performance Dashboard (Power BI)

Route Power BI - Assignment 2. A one-page Power BI dashboard that analyses student exam performance by subject, gender, race/ethnicity and parental education.

![Dashboard](images/dashboard.png)

## Dataset

`study_performance.csv` - 1,000 students, 8 columns, no missing values.

| Column | Type | Notes |
|---|---|---|
| gender | Text | female / male |
| race_ethnicity | Text | group A to group E |
| parental_level_of_education | Text | 6 levels |
| lunch, test_preparation_course | Text | not used in this dashboard |
| math_score, reading_score, writing_score | Whole number | 0-100 |

## Dashboard pages and charts

| Chart | Type | Fields |
|---|---|---|
| Overall Performance | Clustered bar | Subject, Avg Score by Subject |
| Gender Distribution | Pie | gender, Average of Average Score |
| Performance Gap by Race and Ethnicity | Donut | race_ethnicity, Average of Average Score |
| Parental Education Impact | Stacked bar | Parental Education, Student Count, Performance Level |

Plus one card showing the overall average score.

## Data model

### Power Query (calculated column)

```
Average Score = ([math_score] + [reading_score] + [writing_score]) / 3
```

The data has three separate scores and no single "performance" column, so the average of the three subjects is used as the performance measure. Type is Decimal Number and default summarization is set to Average.

### DAX

Helper table for the bar chart:

```
Subject Scores = DATATABLE("Subject", STRING, "Order", INTEGER,
    {{"Math", 1}, {"Reading", 2}, {"Writing", 3}, {"Overall", 4}})
```

Measures:

```
Avg Math    = AVERAGE(study_performance[math_score])
Avg Reading = AVERAGE(study_performance[reading_score])
Avg Writing = AVERAGE(study_performance[writing_score])
Avg Overall = AVERAGE(study_performance[Average Score])

Avg Score by Subject =
SWITCH(SELECTEDVALUE('Subject Scores'[Subject]),
    "Math", [Avg Math], "Reading", [Avg Reading],
    "Writing", [Avg Writing], "Overall", [Avg Overall])

Student Count = COUNTROWS(study_performance)
```

Calculated columns for the stacked bar:

```
Performance Level =
IF(study_performance[Average Score] < 60, "Low",
   IF(study_performance[Average Score] < 80, "Medium", "High"))

Level Order =
IF(study_performance[Average Score] < 60, 1,
   IF(study_performance[Average Score] < 80, 2, 3))

Education Order =
SWITCH(study_performance[parental_level_of_education],
    "some high school", 1, "high school", 2, "some college", 3,
    "associate's degree", 4, "bachelor's degree", 5, "master's degree", 6)

Parental Education = study_performance[parental_level_of_education]
```

Sort by column: `Performance Level` by `Level Order`, and `Parental Education` by `Education Order`.

The performance bands (Low below 60, Medium 60 to under 80, High 80 and above) are my own choice because the assignment does not define them.

## Key results

| Metric | Value |
|---|---|
| Overall average score | 67.77 |
| Math / Reading / Writing | 66.09 / 69.17 / 68.05 |
| Female / Male | 69.57 / 65.84 |
| Group A / B / C / D / E | 62.99 / 65.47 / 67.13 / 69.18 / 72.75 |

Average score rises with parental education: from 63.10 (high school) and 65.11 (some high school) up to 73.60 (master's degree).

## Design

Light theme with a calm purple palette. Pink and blue are used only for the gender chart so those colors keep a single meaning.

| Use | Color |
|---|---|
| Page background | `#F6F5FB` |
| Cards | `#FFFFFF` |
| Female / Male | `#D98BB5` / `#7FA6D9` |
| Bar chart | `#8E86D6` |
| Donut groups A to E | `#6FB7B0`, `#8E86D6`, `#5B6BB5`, `#B39DDB`, `#7C879B` |
| Low / Medium / High | `#D9D4F0`, `#A59CD8`, `#6F63B5` |
| Text | `#2D2B4A` |

## How to open

1. Open `Student_Performance_Dashboard.pbix`.
2. If Power BI asks for the data path, point it to `study_performance.csv` (Transform data, then Data source settings).

## Files

```
.
|-- Student_Performance_Dashboard.pbix
|-- study_performance.csv
|-- images/dashboard.png
|-- Student_Performance_Dashboard_Notes.pdf
`-- README.md
```
