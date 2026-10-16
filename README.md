# HR Workforce Analytics Dashboard (Power BI)
A 3-page Power BI dashboard that analyzes an HR database:
1) Who the workforce is,
2) How employees perform and feel at work
3) How much is invested in training and whether it works.

## Business questions

1. Who works here, and how is the workforce structured (jobs, departments, age, gender, location)?
2. How well do employees perform, and how engaged and satisfied are they?
3. How much does the company spend on training, and does it lead to successful outcomes?

## Dashboard pages

### 1. Workforce Overview

[Workforce Overview](Screenshots/01_workforce.png)

- **443 employees**: 384 active and 59 terminated (turnover rate 13.3%).
- The average age is **55**, and the **60+ group is the largest** (194 employees, about 44% of the workforce).
- **Production is the biggest department** (288 employees), far ahead of IT/IS (68) and Sales (41).
- Employee types are balanced: 162 full-time, 158 temporary and 123 part-time. A large part of the workforce is not permanent.
- Gender split: 59.6% / 40.4%.

### 2. Performance & Engagement

[Performance & Engagement](Screenshots/02_performance.png)

- Average rating **3.0**, engagement **3.1**, satisfaction **3.1**.
- **Work-life balance is the lowest score (2.9)** and slipped slightly from 3.0 in 2022 to 2.9 in 2023.
- Most reviews fall in the middle of the scale: 237 of 443 reviews have a rating of 3.
- IT/IS has the highest engagement (3.3).
- Survey data only covers 2022 and 2023, so the time chart compares those two years.

### 3. Training & Development

[Training & Development](Screenshots/03_training.png)

- Total training cost: **$241.94K** across **443 trainings**.
- The **success rate is 54%** (Completed + Passed = 238 of 443 trainings). 107 trainings failed and 98 are incomplete.
- **Technical Skills has the lowest success rate (40%)**, while Leadership Development (59%) and Customer Service (58%) are the highest.
- Project Management is the most expensive program ($57K) but has an average success rate (55%).
- Spending is split almost evenly between external (50.2%) and internal (49.8%) training.

## Data model

[Data model](Screenshots/04_data_model.png)

`employees` is the central table. It connects to the lookup tables `jobs`, `departments`, `divisions`, `states` and `business_units`. 
The fact tables `performance_reviews`, `employee_surveys` and `employee_training` link back to employees by `employee_id`, and `training_programs` links to the training records.
A separate `Date` table supports time analysis.

Modeling notes:

- Relationships are one-to-many with single-direction filtering where possible, to avoid ambiguous filter paths.
- A `Date` table is related to hire dates, survey dates, review dates and training dates. Several of these relationships are inactive and are activated inside measures with `USERELATIONSHIP`.

## Key DAX measures

``` DAX-List
Avg Rating = AVERAGE('public performance_reviews'[employee_rating])

Avg Engagement = AVERAGE('public employee_surveys'[engagement_score])

Avg Satisfaction = AVERAGE('public employee_surveys'[satisfaction_score])

Avg Work-Life Balance = AVERAGE('public employee_surveys'[work_life_balance_score])

Review Count = COUNTROWS('public performance_reviews')

Total Training Cost = SUM('public employee_training'[training_cost])

Avg Duration (Days) = AVERAGE('public employee_training'[training_duration_days])

Trainings = COUNTROWS('public employee_training')

Successful Trainings =
CALCULATE(
    [Trainings],
    'public employee_training'[training_outcome] IN {"Completed", "Passed"}
)

Success Rate = DIVIDE([Successful Trainings], [Trainings])

Hires =
CALCULATE(
    COUNTROWS('public employees'),
    USERELATIONSHIP('Date'[Date], 'public employees'[start_date])
)

New Hires This Year =
CALCULATE(
    COUNTROWS('public employees'),
    FILTER(
        'public employees',
        YEAR('public employees'[start_date]) = YEAR(TODAY())
    )
)
```

## Design choices

- One dark blue theme across all pages (page `#0B1F33`, cards `#12395C`, bars `#4DA3FF`).
- Orange (`#FFB347`) is used sparingly to mark the number that needs attention, such as the lowest score or the weakest program.
- The same layout on every page: navigation buttons, KPI cards, filters and four charts, so the pages feel like one report.
- Year and Status filters on the Performance and Training pages.

## What I learned

- How to design a clean data model with a central table, lookup tables and a Date table.
- How inactive relationships work when one Date table serves several date columns.
- How to check that numbers reconcile across visuals (for example 443 employees, 443 reviews and 443 trainings).
- How to keep a dashboard consistent: one color palette, one card style, and aligned layouts across pages.

## Data

The data comes from an **HR database that I designed and built in PostgreSQL** in a separate project:
[HR Database Project](https://github.com/Sam-Khach/hr-data-analysis).
That project contains the database structure, the tables and the SQL scripts used to create it.
For this dashboard, I imported the PostgreSQL tables into Power BI Desktop and built the data model, measures and visuals on top of them.

| Table | Contents |

| `employees` | Main table: job, department, status, type, age, gender, location |
| `jobs`, `departments`, `divisions`, `states`, `business_units` | Lookup tables |
| `performance_reviews` | Employee ratings and performance scores |
| `employee_surveys` | Engagement, satisfaction and work-life balance scores |
| `employee_training`, `training_programs` | Training records, cost, duration and outcomes |

**hr-data-analysis** PostgreSQL database → import into Power BI → data model (relationships and a Date table) → DAX measures → 3-page dashboard.
Dashboard built on this database

1. Download `powerbi/HR_Dashboard.pbix`.
2. Open it in **Power BI Desktop** (Windows).

## Author

Samvel Khachatryan | sam.khachnsjsjs@gmail.com
