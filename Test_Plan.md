# Nibl — Test Plan

Date: November 6, 2026

Team: Andrew Ogrin-Cotarlan, Nathan Wiley, Duy Pham, Jack Keller

Traces to: Test_Plan.md

Sources: Assignment 9 v0.2, Assignment 8

## 1. Test scope

This cycle tests the Cincinnati web pilot. A visitor should be able to find a rated dish and interpret its dietary information. A signed-in diner should be able to add a dish or change a review without corrupting another diner’s review or the dish’s rating.

### In scope

| Component | Week 9 IDs | What the team will test |
| --- | --- | --- |
| Firebase Auth and Python API | REQ-01, REQ-11, NFR-02 | Sign-in, public reads, write ownership and HTTPS. |
| Review API and Firestore | REQ-03, REQ-12, REQ- 14, REQ-15, NFR-05 | Create dishes at existing restaurants, validate reviews, update totals atomically and reject missing targets. |
| Discovery API and React | REQ-06, REQ-07, REQ- 08, REQ-09, REQ-16 | Nearby ranking, ZIP fallback, search, cuisine filters, dish details and unsupported areas. |
| Dietary labels and ranking | REQ-02, REQ-10 | Show all three label states and exercise the rule- based recommendation and cold-start paths. |
| Web client and staging service | NFR-01, NFR-03, NFR-04 | Measure response time, first-use task completion and 24-hour search availability. |
| Release constraints | CON-01, CON-02, CON- 03 | Inspect cost settings, data provenance and preference handling. |

### Out of scope and known limitations

REQ-13 (Could) defers photos and Cloud Storage file checks under the $0 budget. REQ-04 was withdrawn pending feasibility review. REQ-05 was withdrawn because the release targets browsers.

Native apps, nationwide coverage, payments, reservations, paid AI/LLMs, verified owner dashboards and notifications are Week 9 Won’t features with no active test IDs.

REQ-10 covers reported dietary information. Kitchen practices and allergy-safety guarantees are outside the pilot’s evidence. REQ-06/07/16 use a limited Cincinnati seed and simulated or denied location. GPS precision, citywide data completeness and traffic beyond the specified load are outside this cycle.

For REQ-01/NFR-02 and NFR-05, test Nibl’s token handling, deployed TLS endpoint and stored results. Firebase sign-in internals, Firestore’s transaction engine and browser geolocation internals remain vendor responsibilities.

## 2. Verification and validation strategy

During each sprint, the team verifies behavior against the Week 9 requirements and API contracts.

Nathan checks API responses, Jack independently recounts Firestore records, and Andrew compares recommendation results with the seed. Duy coordinates QA, links evidence to case IDs and schedules reruns with the task owners.

Before the demo, five first-time diners each use the 360 px site to find a rated dish (REQ-06/08, NFR-

03). A diner with dietary restrictions interprets an unknown label (REQ-10). An owner or owner-role participant locates public feedback (REQ-11). Each works from a task prompt without coaching.

Andrew asks the advisor to review the anonymized outcomes against US-01 through US-03 and record acceptance or a reason to revise.

| Test layer | Approx. count | Coverage focus |
| --- | --- | --- |
| Unit | 40 checks | Rating and length limits, distances, label mapping and ranking rules. |
| Integration | 25 checks | API, token verifier and Firestore interactions, including denied writes. |
| System | 10 scenarios | Browser-to-API flows, TLS, load, concurrency and 24-hour probes. |
| Acceptance | 3 tasks | Find a rated dish, interpret unknown, read public owner feedback. |

Recommendations use deterministic ranking. Hold the candidate dishes, ratings and location fixed, then vary history or a dietary restriction. Compare dish IDs with the expected list once per fixture. Accept zero known conflicts or unknown peanut records for a peanut-restricted user. The pilot evaluates rule behavior, with no model-training or sensor trials.

## 3. Measurable success criteria

| Metric | Target threshold | How measured | Week 9 ID |
| --- | --- | --- | --- |
| Response time | ≥190/200 within 3.000 s | Time 100 searches and 100 detail GETs at 25 concurrent clients. Count only valid 2xx responses. | NFR-01 |
| Metric | Target threshold | How measured | Week 9 ID |
| Search reliability | ≥1,426/1,440 valid probes | Once/min for 24 h. Require HTTP 200 with D01 in a valid result list. Count missed slots as failures. | NFR-04 |
| First-use usability | ≥4/5 finish in ≤120 s | At 360 px, time from task reveal to a rated dish detail page, without help. | NFR-03 |
| Access control | 20/20 denied, zero mutations. TLS ≥1.2 | Replay the 20 negative writes, diff snapshots, then record protocol handshake results. | REQ-01, NFR- 02 |
| Rating accuracy | 0 mismatches per 100 changes | Recount current reviews, sum and count independently. Compare each touched dish. | REQ-03, REQ- 14, NFR-05 |
| Discovery and dietary accuracy | Exact IDs/order in each fixture. 3/3 label states correct. | Compare output with the seed oracle. A single extra dish or unknown shown as safe fails. | REQ-06, REQ- 08, REQ-10 |

## 4. Test cases

The eight Musts are the major requirements for this cycle. A/B suffixes give additional independently scored cases under that entry. All eight Must requirements have three cases. Section 5 defines fixtures, request paths and client settings.

### Sign-in, reviews and recommendations

| ID | Description | Steps | Expected Result | Status |
| --- | --- | --- | --- | --- |
| TC-001 | REQ-01 Valid sign-in | 1. Sign in as U1 through Firebase. 2. Use its ID token to post 4 stars to Blank. Read the stored review. | 2xx. Exactly one review on Blank has author U1 and rating 4. | Not Run |
| TC-001A | REQ-01 Missing token | 1. Omit Authorization. 2. Post 4 stars to Blank. Recount reviews. | 401. Blank still has zero reviews and sum 0. | Not Run |
| TC-001B | REQ-01 Tampered token | 1. Change one signature character in U1’s token. 2. Post the same review to Blank. | 401. No review or rating total changes. | Not Run |
| TC-002 | REQ-02 Cold start | 1. Sign in as fresh U1 with no preferences. 2. Call GET /recommendations with limit 10 on the rank fixture. | 200. Exactly D01–D10 appear in descending popularity order. | Not Run |
| TC-003 | REQ-03 Lower limits | 1. Post 1 star to Blank as U1. Omit comment. 2. Read reviews and totals. | 2xx. One U1 review, count 1, sum 1 and average 1. | Not Run |
| TC-003A | REQ-03 Maximum and edit | 1. Post 1 star to Blank as U1. 2. Post 5 stars with 500 “x” characters as U1 to Blank. Read back. | Both 2xx. One U1 review with exactly 500 characters, sum 5 and count 1. | Not Run |
| ID | Description | Steps | Expected Result | Status |
| TC-003B | REQ-03 Invalid inputs | 1. As U1, post to Blank with 0 stars, then 6 stars. 2. Post 3 stars with 501 “x” characters. Read totals. | All three return 400. Blank has zero reviews and sum 0. | Not Run |

### Execution record

For each row, record the build, seed version, tester, timestamp and actual outcome beside the evidence.

A row with several inputs passes only when every input gives its expected result.

### Review request example

For TC-003, send POST /api/v1/dishes/Blank/reviews with U1’s ID token in the Authorization header

(Bearer scheme), Content-Type application/json and the body {"rating": 1}. Use the admin read-only export to confirm that U1 owns the one stored review. A success message in the browser alone cannot establish the stored count.

### Test cases continued — nearby discovery and dietary labels

| ID | Description | Steps | Expected Result | Status |
| --- | --- | --- | --- | --- |
| TC-004 | REQ-06 Rank and cap | 1. Load the base fixture and allow the fixed origin. 2. Open nearby dishes and record ordered IDs. | Exactly D01–D10 in that order. Unrated Blank and outside Far are absent. | Not Run |
| TC-004A | REQ-06 Five-mile limit | 1. Supply only two rated candidates, at 5.000 and 5.001 mi, to the nearby filter. 2. Run the filter with radius 5 mi. | Only the 5.000 mi candidate is returned. | Not Run |
| TC-004B | REQ-06 No rated dishes | 1. Seed only Blank at the fixed origin. 2. Refresh the nearby list. | Zero nearby ranked dishes. No Unrated item occupies a slot. | Not Run |
| TC-005 | REQ-07 ZIP fallback | 1. Deny browser location and reload. 2. Enter 45202 in the location prompt. | The supported ZIP produces the seeded nearby list D01–D10. | Not Run |
| TC-006 | REQ-08 Rated detail | 1. Open D01 from nearby results. 2. Compare name, restaurant, average and reviews with the seed. | “Chili Bowl”, “River Cafe”, 5 stars and all 10 seeded reviews appear. | Not Run |
| TC-006A | REQ-08 Unrated detail | 1. Open Blank directly via its dish detail URL. 2. Read the restaurant, rating and reviews. | “Plain Rice”, “River Cafe”, “Unrated” and zero reviews. | Not Run |
| TC-006B | REQ-08 Different dish | 1. Open D01, then navigate to D02. 2. Compare the new detail with D02’s seed record. | “Bean Bowl”, “Hill Cafe”, 4.9 stars and only D02’s 10 reviews. | Not Run |
| ID | Description | Steps | Expected Result | Status |
| TC-007 | REQ-09 Search and filter | 1. Search “Chili Bowl”, then “River Cafe”. 2. Apply Mexican to River Cafe. Clear filters and search “zzzz”. | Only D01 for the dish query. River Cafe matches its seed list. Mexican returns zero. Last query shows “No dishes found”. | Not Run |
| TC-008 | REQ-10 Conflict | 1. Set U1’s peanut restriction. 2. Open D01 and inspect its peanut label. | One peanut conflict is shown for D01. | Not Run |
| TC-008A | REQ-10 Reported absent | 1. Set U1’s peanut restriction. 2. Open D02 and inspect its peanut label. | Peanut is labeled “reported absent” for D02. | Not Run |
| TC-008B | REQ-10 Missing information | 1. Set U1’s peanut restriction. 2. Open Blank, which lacks a peanut record. | Peanut is “unknown”. No safe or reported- absent claim appears for peanut. | Not Run |

### Test cases continued — contributions, security and load

| ID | Description | Steps | Expected Result | Status |
| --- | --- | --- | --- | --- |
| TC-009 | REQ-11 Public feedback | 1. Open D01 as visitor, then in the owner-role session. 2. As owner, attempt a direct Firestore edit to another author’s review. | Both sessions see 10 reviews. Write is denied. Original author, text and rating stay intact. | Not Run |
| TC-010 | REQ-12 Create dish | 1. As U1, POST /dishes with restaurantId R1 and name “Test Soup”. 2. Read the new dish by its returned ID. | 2xx. Exactly one new dish named Test Soup links only to R1. | Not Run |
| TC-010A | REQ-12 Same dish name | 1. As U1, create “Test Soup” once at R1 and once at R2. 2. Read each returned ID. | Two distinct IDs. Each dish references exactly its submitted restaurant. | Not Run |
| TC-010B | REQ-12 Missing restaurant | 1. As U1, POST /dishes with restaurantId Missing and name “Test Soup”. 2. Compare the dish collection before and after. | 404. Zero dishes are created or modified. | Not Run |
| TC-012 | REQ-14 Edit review | 1. Post 2 stars to Blank as U1. 2. Post 4 stars to Blank as U1. Read the review and totals. | One current review for U1. Sum 4, count 1 and average 4. | Not Run |
| TC-013 | REQ-15 Missing target | 1. Snapshot totals. As U1, post 5 stars to dish Missing. 2. Compare review and dish snapshots. | 404. No review created and zero differences in any dish total. | Not Run |
| ID | Description | Steps | Expected Result | Status |
| TC-014 | REQ-16 Unsupported ZIP | 1. Deny location. Enter 10001 in the ZIP prompt. 2. Inspect the message and list. | Unsupported-area message is visible and the list contains zero dishes. | Not Run |
| TC-015 | NFR-01 Search/detail load | 1. Run 25 clients in parallel, each alternating GET /dishes and GET /dishes/D01 four times each. 2. Time all 200 requests end to end. | At least 190 valid 2xx responses complete in ≤3.000 s. | Not Run |
| TC-016 | NFR-02 Secure transport | 1. Probe the staging API host with TLS 1.0, 1.1 and a supported version ≥1.2. Enable each version on the probe client. 2. GET /dishes over the accepted version. | 1.0 and 1.1 handshakes fail. TLS ≥1.2 succeeds with a valid certificate and HTTP 200. | Not Run |
| TC-016A | NFR-02 Denied API writes | 1. Use both POST routes. For each, send absent, malformed, expired and wrong-project tokens twice. 2. Diff snapshots after all 16 attempts. | 16/16 return 401. Zero dish, review or total changes. | Not Run |
| TC-016B | NFR-02 Direct writes | 1. As visitor and then as U2, try direct Firestore writes to D01’s totals and U1’s seeded review on AuthDish. 2. Compare snapshots across four attempts. | 4/4 denied by Firestore rules. Zero mutations. With TC-016A, 20/20 denied. | Not Run |

### Test cases continued — pilot, reliability and rating accuracy

| ID | Description | Steps | Expected Result | Status |
| --- | --- | --- | --- | --- |
| TC-017 | NFR-03 First-time task | 1. Give each of five new diners a fresh 360 px session at the fixed origin. 2. Say “Find and open a rated dish near you.” Time each attempt without help. | At least 4/5 reach a detail page with a numeric rating within 120 s. | Not Run |
| TC-018 | NFR-04 24-hour search | 1. Schedule GET /dishes?query=Chili%20Bowl once per minute for 24 h. 2. Count HTTP 200 responses with valid JSON containing D01 by the next slot. | At least 1,426 of 1,440 slots succeed. Errors, timeouts and missed slots count against the threshold. | Not Run |
| TC-019 | NFR-05 Parallel inserts | 1. Release 100 distinct users together to post 4 stars to Blank. 2. Await completion. Independently recount reviews and totals. | 100 commits, 100 reviews, sum 400 and average 4. Zero stored/recount differences. | Not Run |
| TC-019A | NFR-05 Parallel edits | 1. Seed 100 users with 2 stars each on Blank. Release all 100 to change to 5. 2. Recount after completion. | 100 commits. Count 100, sum 500 and average 5. Zero differences. | Not Run |
| TC-019B | NFR-05 Same-user contention | 1. Release 100 concurrent 3-star posts to Blank as U1. 2. Retry HTTP 409 responses serially, at most three times each. Recount. | 100 logical writes succeed. Exactly one U1 review, sum 3 and count 1. Zero differences. | Not Run |
| ID | Description | Steps | Expected Result | Status |
| TC-020 | CON-01 Free-tier budget | 1. Inspect the staging and pilot cost settings. 2. Compare planned request/storage usage with free allowances before load tests. | Projected charge $0 and zero paid resources required for the run. | Not Run |
| TC-021 | CON-02 Seed provenance | 1. List each seed source and its permission or synthetic-data note. 2. Compare the inventory to the seed manifest. | 100% of sources accounted for. Zero proprietary or unapproved imports. | Not Run |
| TC-022 | CON-03 Preference handling | 1. Using the rank fixture, set U1’s peanut restriction and preference for D02’s cuisine. 2. Request 10 recommendations. Inspect build settings. | D02 appears. No conflict or unknown- peanut dish appears. Photo upload is disabled. | Not Run |

TC-011 is reserved for deferred REQ-13. It is excluded from this cycle’s execution count.

## 5. Test environment

Proposed reproducible baseline. Before the first run, record the Git commit, lockfile hashes, exact build numbers, seed export hash, Firestore rules revision, API host and UTC run window. Use separate

Firebase projects for staging and pilot. Keep tokens out of evidence.

| Environment | Hardware, software and configuration |
| --- | --- |
| Development | x86-64 laptop or VM, 4 vCPU/8 GB RAM, Ubuntu 24.04 LTS, Python 3.12, Node 22 and Chrome 142 at 1280 × 800. Pin React and Firebase SDK/CLI versions in the run manifest. Use local Auth and Firestore emulators, deterministic fixtures and loopback networking. |
| Staging | Same commit and dependencies, Firebase Hosting, isolated Auth and Firestore, managed Python HTTPS API. Use the same Linux client on a fixed 20 Mbps down/10 Mbps up link, 40 ms round-trip latency, zero injected loss. Disable browser caching for loads. Record region and runtime before deployment. |
| Pilot | Release candidate on a separate HTTPS endpoint. Use Chrome 142, a 360 × 800 CSS-pixel viewport at 100% zoom and the staging network profile. Give each participant a clean browser profile. Override location to the fixture or deny permission as the case directs. |

### Seed and request conventions

Base fixture: set the origin and test centroid of ZIP 45202 to (39.1031, −84.5120). Put R1 River Cafe and R2 Hill Cafe at the origin. Assign odd D01–D12 to R1 (American) and even IDs to R2 (Mexican).

D01 is Chili Bowl, D02 is Bean Bowl. Name D03–D12 by ID. For dish number n, seed ten reviews summing to 51 − n, so averages run from 5.0 to 3.9. Label D01 peanut conflict and D02 reported absent. All other peanut values are unknown.

Blank is Plain Rice at R1, with no reviews. Far is a 5-star dish at (39.2031, −84.5120). The recommendation-only rank fixture contains just D01–D12, with equal review counts and popularity ordered by average. U1 and U2 are test accounts separate from the ten seed authors. The owner role uses U2. Only TC-016B adds AuthDish and U1’s review. “Missing” is an unused ID. Create 100

disposable accounts for concurrency and release requests from a common start barrier.

Prefix API paths with /api/v1. To post a review, POST /dishes/{id}/reviews with integer rating, optional comment and the stated Bearer token. Compare sorted Firestore exports before/after denied writes. For concurrency, group persisted reviews by dish and author, then recompute count, sum and average.

Record every non-2xx response and retry.

## 6. Pass and fail criteria

Pass a row only when all observed outcomes equal its expected results, including persisted state. Fail an executed row for any threshold miss or unexpected result. Mark Blocked if a prerequisite prevents a valid run and Not Run if execution has yet to start. Attach the actual result and evidence to each failure or blocker.

Pass the plan only when all 37 in-scope rows pass on the candidate, every Section 3 threshold is met, stakeholder tasks meet Section 8 criteria and sign-off is recorded. Any failed gate fails the candidate.

Any Blocked or Not Run row leaves the decision pending. Rerun affected cases after a fix.

## 7. Test schedule and milestones

M1–M4 established the constraints, requirements, architecture, schema and API contracts. The dates and milestone owners below come from Assignment 8. Week 9 specifies Firebase sign-in for the earlier

AES milestones (M5/M10) and defers the notifications listed under M11.

| Milestone | Test activity | Owner | Evidence |
| --- | --- | --- | --- |
| M5 · Nov 9, 2026 API contracts | Unit-test endpoint parameters, auth error responses and schema boundaries. Track the Nov 12 Firebase decision (A-01). | Jack | Unit report and auth decision record. |
| M6 · Nov 16 M7 · Nov 23 M8 · Nov 30, 2026 | M6: check test effort and the $0 budget. M7: audit requirement-to- case links and risks. M8: submit the plan for advisor review. | M6 Nathan M7 Andrew M8 Jack | Effort matrix, audited test matrix and dated planning sign-off. |
| M9 · Jan 31, 2027 Database and API setup | During T2–T4 and T6–T7, verify seed data, schema and authenticated connections. Run T5 permission and concurrency checks Jan 25–Feb 5. | Duy and Andrew Jack owns T5. | Seed hash, rules report and before/after recount. |
| M10 · Feb 28, 2027 Working REST endpoints | During T8–T9 and T17–T19, run unit and integration cases for discovery, dietary filters, fallback and review totals. | Jack and Nathan Andrew owns ranking. | API collection results, ranked IDs and review diffs. |
| M11 · Mar 31, 2027 Connected React workflows | Run T10/T20 API and ranking checks Mar 1–12, then T15 browser system tests Mar 15–23. Run the three acceptance tasks before the M11 review. | Duy, Nathan, Luke | API log, browser results, five task times and stakeholder notes. |
| Milestone | Test activity | Owner | Evidence |
| M12 · Apr 16, 2027 Final system validation | Before the final review/demo, run load and failure cases. Start the 24-hour probe at least 48 hours before review. Rerun fixes and review acceptance evidence. | Andrew and Jack | Latency and probe logs, defect closures, release commit and sign-off. |

## 8. Definition of done

### Feature checklist

☐  Every applicable case passes on the recorded commit, including a boundary or failure case. A

teammate reviews the implementation and its observed results.

☐  The owner updates the requirement-to-test link, API or UI documentation, seed and evidence. Each open defect has severity, owner and disposition.

### Project checklist

☐  All Section 6 gates pass. No open defect allows unauthorized writes, wrong totals, a misleading dietary label or a blocked core workflow.

☐  At least four of five diners complete the timed task. The dietary participant explains that unknown means the information is missing, and the owner independently locates one seeded public review.

Record one successful observation for each of those two tasks.

☐  QA and the coordinator review the candidate. The advisor records dated acceptance. Attach the signed decision and release commit. Sign-off is pending until that record exists.
