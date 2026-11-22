# Dare Careers – Learner Progress & Performance Dashboard (Power BI)

A Power BI dashboard that tracks learner engagement and outcomes for Dare Careers'
10-week training programme. The data covers **one cohort** (5 Aug – 11 Oct 2024)
with **two learning tracks**, **Power BI** and **AWS Cloud**. The model is built so
more cohorts can be added later.

It brings together daily Zoom attendance, daily participation, weekly lab and quiz
scores, and graduation/certification status. The original brief is in
[`docs/project_brief.md`](docs/project_brief.md).

## Dashboard pages

| Page | Audience | What it shows |
|---|---|---|
| **1. Overall Performance** | Programme stakeholders | Cards: total learners, graduations, certifications, dropouts. Bar charts by cohort and track: graduation, certification and dropout rates; average attendance (% of class time), participation and assessment score |
| **2. Learner Insights** | Trainers & programme managers | Cards: labs completed, average labs per learner, total hours in class, attendance rate, participation rate, average assessment score. A learner table with each learner's track, certification status and key metrics |

**Slicers:** Page 1 has Cohort, Track, Certification Type and Learner Status. Page 2 has Cohort,
Track, Month, Week, Learner Status and Program Status.

### Page 1 – Overall Performance

![Page 1 – Overall Performance](screenshots/Page%201.png)

### Page 2 – Learner Insights

![Page 2 – Learner Insights](screenshots/Page%202.png)

## Headline results (Cohort 1)

| Metric | Power BI | AWS Cloud | Overall |
|---|---|---|---|
| Learners | 100 | 40 | 140 |
| Graduation rate | 86.0% | 82.5% | 85.0% |
| Certification rate | 68.0% | 60.0% | 65.7% |
| Dropout rate | 0.0% | 17.5% | 5.0% |
| Average attendance (% of class time) | 87.5% | 45.3% | 75.5% |
| Attendance rate (% of sessions attended) | 100.0% | 79.3% | 94.1% |
| Participation rate | 85.1% | 89.2% | 86.2% |
| Average assessment score (0–100) | 50.4 | 51.8 | 50.8 |
| Labs completed | 987 | 396 | 1,383 |

### Observations

- **All 7 dropouts are in the AWS Cloud track.** Every Power BI learner attended every
  session, so none of them meets the "incomplete attendance" condition.
- **Power BI learners always show up but don't always stay.** Their session rate is 100%,
  but they attend 87.5% of class time on average, typically leaving before the 2-hour
  session ends.
- **Cloud participation is higher than Cloud attendance.** There are 1,784 participation
  records but only 1,585 attended sessions (>30 min), so at least 199 times a learner was
  recorded as participating on a day they spent 30 minutes or less in the call. This is
  worth checking with the trainers: either short visits involved real engagement, or
  participation was recorded loosely.
- **Assessment scores are similar across tracks** (about 50/100) and don't explain the
  difference in outcomes between tracks.

## Data

| Dataset | Format | Power BI track | AWS Cloud track | Learner key |
|---|---|---|---|---|
| Zoom attendance | 50 daily CSVs in `Week 1`–`Week 10` folders | 100 learners/day | 40 learners/day | email |
| Participation | One row per day, learner names comma-separated | 50 class days | 50 class days | full name |
| Labs & quizzes | Two sheets (`Labs`, `Quizzes`), wide: Week 1–10 columns | 100 learners | 40 learners | email |
| Status | Graduation and certification status | 100 learners | 40 learners | email |

**Data quirk:** the two tracks record Zoom session time differently. Power BI uses
12-hour clock times (`06:02 PM`) and `h:mm:ss` durations (`1:46:00`). AWS Cloud uses
24-hour times (`18:09`) and durations in whole minutes (`66`). Power Query reads every
file as text and converts both formats into a single `DurationMinutes` column.

## Data model

A star schema with four dimensions and three fact tables. All relationships are
one-to-many with single-direction filtering.

```
            Cohorts
               │
   Date ── Learners ── Weeks
     │      │  │  │      │
     └── Attendance · Participation · Assessment ──┘
```

| Table | Type | Grain | Built from |
|---|---|---|---|
| `Learners` | Dimension | One row per learner | Zoom roster + status files; includes cohort, certification type and the dropout flag |
| `Cohorts` | Dimension | One row per cohort | Entered manually (name, start and end date, Program Status) |
| `Date` | Dimension (DAX, marked as date table) | One row per day | Calendar from the earliest cohort start to the latest cohort end |
| `Weeks` | Dimension (DAX) | One row per programme week | Distinct weeks found in the fact tables |
| `Attendance` | Fact | Learner × class day | Zoom CSVs |
| `Participation` | Fact | Learner × day participated | Participation workbooks |
| `Assessment` | Fact | Learner × week × lab/quiz | Labs & quizzes workbooks, unpivoted |

**Power Query highlights**

- A `DataRoot` parameter holds the data folder path, so changing one value points every
  query at a new location.
- Staging queries (`ZoomStaging`, `Status`, `AttendanceSummary`, `LearnerCohortStart`) are
  not loaded into the model. They do shared work once.
- Programme week is calculated **per learner from their cohort's start date**, so the
  same calendar date can be a different week for a different cohort.
- Every email key is trimmed and lowercased before joins.

**DAX measures** are kept in a `_Measures` table, in display folders
(Learner Outcomes, Attendance, Participation, Assessments, Helpers). Measures with
assumptions have descriptions that show up as tooltips in the field list.

## Business rules and assumptions

| # | Rule / assumption | Implementation |
|---|---|---|
| 1 | A learner **attended** a session only if they spent **more than 30 minutes** in the Zoom call (from the brief) | `Attendance[Attended] = DurationMinutes > 30` |
| 2 | **Dropout** = Non Graduate **and** Not Certified **and** incomplete attendance (client update). Every learner has a Zoom record for all 50 days, so *incomplete* is interpreted as **missed at least one session** (*pending client confirmation*) | `Learners[Is Dropout]`: sessions attended < sessions held |
| 3 | **Certification type** isn't in the data, so it's derived from track: Power BI → Power BI, AWS Cloud → AWS | `Learners[Certification Type]` |
| 4 | A **lab counts as completed** when its score is above 0. A score of 0 is treated as not submitted, but still counts toward average scores | `Labs Completed` measure |
| 5 | Each **assessment is dated** to the Friday of its programme week. So Week 9 (30 Sep – 4 Oct) assessments fall in October under the Month slicer | `Assessment[Date]` |
| 6 | **Hours and class-time %** count only sessions over 30 minutes. A class lasts **120 minutes** (18:00–20:00) | `Total Hours in Class`, `Class Time Attended %`, `Session Length (min)` |
| 7 | **One cohort, two tracks** (client update). Bar charts use Cohort on the axis and Track as the legend, so new cohorts appear without a redesign | `Cohorts` table |

Two attendance measures are shown on purpose:
**Average Attendance (% of class time)** on Page 1 answers *"do learners stay?"*, and
**Attendance Rate (% of sessions attended)** on Page 2 answers *"do learners show up?"*

## Repository structure

```
.
├── data/
│   └── raw/                                   # Source files as received
│       ├── powerbi/                           # labs_quizzes/, participation/, status/
│       ├── aws_cloud/                         # same layout
│       ├── Zoom Attendance PowerBI training/  # Week 1 … Week 10 / DD-Mon-YYYY.csv
│       └── Zoom Attendance Cloud Training/    # Week 1 … Week 10 / DD-Mon-YYYY.csv
├── docs/
│   └── project_brief.md                       # Original project requirements
├── powerbi/
│   ├── DareCareers.pbip                       # Open this in Power BI Desktop
│   ├── DareCareers.SemanticModel/             # Queries, tables, relationships, measures (TMDL)
│   └── DareCareers.Report/                    # Report pages and visuals
└── screenshots/                               # Page 1.png, Page 2.png
```

## How to open and refresh

1. Clone the repository.
2. Open `powerbi/DareCareers.pbip` in Power BI Desktop.
3. **Transform data → Manage Parameters → `DataRoot`**: set it to your local
   `…\dare-careers-analysis\data\raw` folder (no trailing backslash).
4. **Close & Apply**, then **Refresh**.

## Tools

- Power BI Desktop, saved as a Power BI Project (`.pbip`) so changes are readable in Git
- Power Query (M) and DAX
