# Dare Careers – Learner Progress & Performance Dashboard (Power BI)

A Power BI dashboard that tracks learner engagement and outcomes for Dare Careers'
two 10-week training tracks: **Power BI** and **AWS Cloud**.

It covers daily Zoom attendance, daily participation, weekly lab and quiz scores,
and graduation/certification status. The full brief is in
[`docs/project_brief.md`](docs/project_brief.md).

## Dashboard pages

| Page | Audience | What it shows |
|---|---|---|
| 1. Overall Performance Metrics | Program stakeholders | Graduation, certification and dropout rates; average attendance, participation and assessment scores |
| 2. Detailed Learner Insights | Trainers & program managers | Per-learner table of attendance, participation and scores, plus lab and class-hour summaries |

## Repository structure

```
.
├── data/
│   └── raw/                      # Source files as received (renamed only)
│       ├── powerbi/
│       │   ├── labs_quizzes/     # Weekly lab & quiz scores (wide: Week 1–10)
│       │   ├── participation/    # One row per day, comma-separated learner names
│       │   ├── status/           # Graduation & certification status per learner
│       │   └── zoom_attendance/  # Daily Zoom logs, Week 1–10 (to be added)
│       └── aws_cloud/            # Same layout as powerbi/
├── docs/
│   └── project_brief.md          # Original project requirements
├── powerbi/                      # Power BI Project (.pbip) files
└── screenshots/                  # Dashboard screenshots for this README
```

## Datasets

| Dataset | Power BI track | AWS Cloud track | Learner key |
|---|---|---|---|
| Labs & quizzes | 100 learners × 10 weeks | 40 learners × 10 weeks | email |
| Participation | 50 class days (05 Aug – 11 Oct 2024) | 50 class days | full name |
| Status | 100 learners | 40 learners | email |
| Zoom attendance | *pending* | *pending* | – |

Original file names were made consistent when added to the repo:

| Original | In repo (`data/raw/…`) |
|---|---|
| `Labs___Quizes_PowerBI_Training.xlsx` | `powerbi/labs_quizzes/labs_quizzes_powerbi.xlsx` |
| `Participation_records_PowerBI_training.xlsx` | `powerbi/participation/participation_powerbi.xlsx` |
| `Status_of_Participant_PowerBI_training.xlsx` | `powerbi/status/status_powerbi.xlsx` |
| `labs_and_quizzes_cloud_training.xlsx` | `aws_cloud/labs_quizzes/labs_quizzes_aws_cloud.xlsx` |
| `participation_cloud_training.xlsx` | `aws_cloud/participation/participation_aws_cloud.xlsx` |
| `participant_status_cloud_traiining.xlsx` | `aws_cloud/status/status_aws_cloud.xlsx` |

## Business rules

- A learner counts as **attended** for a session only if they spent **more than 30 minutes** in the Zoom call.

## Project roadmap

- [x] Phase 1 – Repository setup and raw data
- [ ] Phase 2 – Data preparation in Power Query
- [ ] Phase 3 – Data model and relationships
- [ ] Phase 4 – DAX measures
- [ ] Phase 5 – Page 1: Overall Performance Metrics
- [ ] Phase 6 – Page 2: Detailed Learner Insights
- [ ] Phase 7 – Documentation and screenshots

## Tools

- Power BI Desktop, saved as a Power BI Project (`.pbip`) so changes are readable in Git
- Power Query (M) and DAX
