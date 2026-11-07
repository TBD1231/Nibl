# Nibl — Requirements Document

Date: November 5, 2026

Team: Andrew Ogrin-Cotarlan, Nathan Wiley, Duy Pham, Jack Keller

Traces to: Technical_Specification.md v0.2

Sources: D0, D1, D2, and Assignments 3, 4 and 8.

## 1. Version and change log

| Version | Date | Change | Author |
| --- | --- | --- | --- |
| 0.1 | 2026-11-05 | Consolidated earlier design work into requirements. | Andrew Ogrin- Cotarlan |
| 0.2 | 2026-11-05 | Simplified language and layout; requirements and IDs unchanged. | Jack Keller |

## 2. Purpose and scope

Nibl helps people choose what to order by reviewing individual dishes, not just restaurants.

The first release is a website focused on Cincinnati restaurants. Visitors can discover and read about dishes. Signed-in users can add dishes and rate/review them. Restaurant owners can read public feedback. Dietary information is shown with warnings and an unknown label when data is missing.

Included: local discovery, search, dish pages, reviews, ratings, account sign-in, adding dishes, and dietary labels. Lower-priority features: personalized recommendations and dish photos.

Excluded: mobile apps, nationwide coverage, restaurant payments, and owner management dashboards.

## 3. Stakeholders and User Goals

| Role | Story | What they need |
| --- | --- | --- |
| Diner / Food Connoisseur | US-01 | Find highly rated dishes, read feedback, and write reviews. |
| Restaurant Owner | US-02 | Read dish-specific feedback to improve menu items. |
| Person with dietary restrictions | US-03 | See potential dietary conflicts before ordering. |
| Project team and advisor | D0–D2 / Week 8 | A feasible, testable plan for the website. |

## 4. Glossary

| Term | Meaning |
| --- | --- |
| Diner | Person browsing or reviewing food; also called Food Connoisseur in US-01. |
| Restaurant Owner | Operator reading public feedback; no special edit rights are assumed. |
| Person with dietary restrictions | User who needs ingredient or suitability warnings. |
| Dish / menu item | A specific food offering at one restaurant. |
| Review | A diner’s 1–5 star rating and optional written feedback for a dish. |
| Average rating | Rating average; shown as Unrated when no reviews exist. |
| Dietary tag | Reported dietary information: conflict, reported absent, or unknown. |
| Recommendation | A ranked set of dishes based on ratings and available preferences. |
| Authenticated user | User whose sign-in token has been verified. |
| Pilot | Initial Cincinnati-area release with limited restaurant data. |
| MoSCoW | Must, Should, Could, and Won’t priorities for the release. |

## 5. Functional requirements

| ID | Requirement statement (pass/fail condition) | Priority | Source | Verify |
| --- | --- | --- | --- | --- |
| REQ-01 | The system shall use Firebase Authentication for sign-in and reject 100% of writes with invalid/missing tokens. | Must | US-01, D2 | T |
| REQ-02 | The system shall return up to 10 recommended dishes, using a popular-dish fallback for users with no rating history. | Should | US-01/03, D2 | T |
| REQ-03 | The system shall accept 1–5 star ratings and optional 0–500 character reviews; store no more than one current review per user/dish. | Must | US-01, D1/D2 | T |
| REQ-06 | The system shall show up to 10 rated dishes within 5 miles, sorted by rating; omit unrated dishes from this list. | Must | US-01, UC-01 | T/D |
| REQ-07 | The system shall accept a supported 5-digit ZIP code when browser location is unavailable. | Should | US-01, UC-01 | T/D |
| REQ-08 | The system shall show the selected dish’s name, restaurant, rating or Unrated, and available reviews. | Must | US-01, UC-01 | T/D |
| REQ-09 | The system shall support dish/restaurant search and cuisine filtering; show No dishes found for zero results. | Should | US-01, UC-01 | T/D |
| REQ-10 | The system shall show dietary labels as conflict, reported absent, or unknown; never treat unknown as safe. | Must | US-03 | T/I |
| REQ-11 | The system shall let visitors and restaurant owners read public reviews but not modify someone else’s review. | Should | US-02, D2 | T |
| REQ-12 | The system shall let a verified user add a dish to one existing restaurant; reject invalid restaurant IDs. | Must | US-01, D0/D1 | T |
| REQ-13 | The system shall if photos are included, display ready photos and reject non- JPEG/PNG files or files over 5 MiB. | Could | US-01, D1 | T |
| REQ-14 | The system shall update an existing review without duplicating it; change rating totals atomically. | Should | US-01, D2 | T |
| REQ-15 | The system shall return HTTP 404 for a nonexistent dish review target without changing rating totals. | Should | US-01, D1/D2 | T |
| REQ-16 | The system shall show an unsupported- area message and zero out-of- area results. | Should | US-01, UC-01 | T/D |

Legend:

- T means automated/manual test - D means needing demonstration - I means needing further inspection.

Withdrawn Requirements:

- REQ-04: Team is reconsidering feasibility - REQ-05: Mobile clients beyond scope

## 6. Non-functional requirements

| ID | Requirement statement (pass/fail condition) | Priority | Source | Verify |
| --- | --- | --- | --- | --- |
| NFR-01 | The system shall complete 95% of search/detail requests within 3 seconds at 25 concurrent users (at least 200 seeded requests). | Should | US-01, D0 | T |
| NFR-02 | The system shall use HTTPS/TLS 1.2+ and reject 100% of unauthorized writes across at least 20 security tests. | Must | US-01, D2 | T/I |
| NFR-03 | The system shall at 360 px width, let 4 of 5 first- time users open a rated dish within 2 minutes. | Should | US-01, UC-01 | T/D |
| NFR-04 | The system shall return a defined result for 99% of one-minute search probes during a 24-hour staging test. | Should | US-01, D0 | T |
| NFR-05 | The system shall after 100 concurrent review changes, have zero differences between stored rating totals and a fresh recount. | Must | US-01, D2 | T |

## 7. MoSCoW summary

- Must: REQ-01, 03, 06, 08, 10, 12; NFR-02, 05. Core discovery, contribution, security, and correct ratings.

- Should: REQ-02, 07, 09, 11, 14, 15, 16; NFR-01, 03, 04. Useful improvements after the core workflow.

- Could: REQ-13. Photo upload/display if time and cost allow.

- Won’t (this release): Native mobile apps, nationwide data, payments/reservations, paid

AI/LLMs, verified owner dashboards, notifications, and guarantees that a dish is allergy-

safe.

## 8. Constraints

| ID | What it forces | What it rules out |
| --- | --- | --- |
| CON-01 | $0 Budget; Free-tier cloud hosting with restricted scaling capabilities | Scaling the platform to accommodate more than what the free tier offers. |
| CON-02 | Data extraction methods must comply with legal regulations on a per website basis | Data from websites with strict terms of service. Any proprietary information. |
| CON-03 | User preference data is prioritized for recommendation algorithm and dietary restrictions | High resolution food photo uploads due to limited free- tier cloud hosting capacity (as discussed in CON-01). |

## 9. Assumptions and Open Questions

| Item | What needs confirmation | Owner | Target date |
| --- | --- | --- | --- |
| A-01 | Use Firebase Authentication instead of D1’s AES password-decryption concept. | Nathan / team | 2026-11-12 |
| A-02 | Confirm team members and role assignments; Week 8 differs from D0. | Team | 2026-11-12 |
| A-03 | Approve priorities, numeric test targets, and recommendation scope. | Team / advisor | 2026-11-16 |
| A-04 | Confirm Cincinnati seed data, supported ZIP codes, and photo costs. | Jack / Team | 2026-11-16 |
| A-05 | Check exact Week 3 constraints and diagram filenames. | Nathan | 2026-11-12 |
| A-06 | Arrange advisor review; no feedback has yet been recorded. | Team | 2026-11-16 |

## 10. Advisor Review

### Status: Pending Advisor Review

Review date: Not yet scheduled/confirmed.

What was discussed: None recorded yet.

Changes resulting from review: None recorded yet.

When the team meets with the advisor, the above section will be updated to reflect that.
