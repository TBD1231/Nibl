# Nibl — Technical Specification Document

Date: November 6, 2026

Team: Andrew Ogrin-Cotarlan, Nathan Wiley, Duy Pham, Jack Keller

Traces to: Requirements_Document.md v0.2

Sources: D0, D1, D2, and Assignments 3 and 8.

## 1. Version and change log

| Version | Date | Change | Author(s) |
| --- | --- | --- | --- |
| 0.1 | 2026-11-05 | Collected earlier architecture, interface, and Draft for team traceability work. | Andrew Ogrin- Cotarlan |
| 0.2 | 2026-11-06 | Shortened descriptions and tables; kept all Draft for team 24 active IDs and test links. | Jack Keller |

## 2. Architecture summary

Nibl uses the three-tier architecture selected in D0:

1. React frontend: Displays dish search, ratings, reviews, and account screens.

2. Python API backend: Processes requests, checks sign-in tokens, validates data, and performs database updates.

3. Firestore database: Stores restaurants, dishes, user profiles, reviews, and rating totals.

A recommendation module runs inside the backend and ranks dish results. Firebase Authentication handles sign-in. Optional photo files are stored separately in Cloud Storage. The frontend does not write directly to Firestore.

Search flow: Browser → HTTPS API → Firestore → ranking module → JSON response → browser.

Review flow: Signed-in browser → API validation → Firestore transaction (review plus rating totals) → confirmation. Errors are returned to the frontend, which shows an error message to the user.

Deployment: React via Firebase Hosting; Python via a compatible managed HTTPS runtime; Firestore and Firebase Authentication as managed services.

Diagrams:

- D0_High_Level_Design - D1_Detailed_System_Design - D2_Detailed_System_Design

## 3. Interfaces and data formats

### 3.1 Component boundaries

| ID / boundary | Inputs → outputs | Data format / protocol | Error behavior |
| --- | --- | --- | --- |
| I1: Frontend ↔ API | Searches, filters, reviews → results, details, confirmations | JSON over REST/HTTPS (/ api/v1); optional multipart photo upload | 400 bad data, 401/403 authorization, 404 not found, 409 conflict, 500/503 failure; UI shows error and keeps review draft. |
| I2: Backend ↔ Firestore | Document lookups, validated writes → documents/ transaction results | Firestore SDK, document objects | 404 missing item, 409 conflict, 503 unavailable; never report unsaved changes as saved. |
| I3: Backend ↔ Ranking | Dish candidates, restrictions, preferences → ranked list | In-process Python function; typed objects/arrays | Empty input → empty list; bad input → validation error; do not ignore dietary conflicts. |
| I4: Browser ↔ Firebase Auth | Sign-in information → ID token / error | Firebase SDK over HTTPS | Invalid login → sign-in error; application does not store passwords. |
| I5: Backend ↔ Token verifier | Bearer token → verified user ID | Firebase Admin SDK | Invalid token → 401, forbidden action → 403, outage → 503. |
| I6: Backend ↔ Photo storage | JPEG/PNG bytes → photo record / object key | Cloud Storage SDK + Firestore metadata | Over 5 MiB → 413, wrong file type → 415, storage error → 503; photos are optional. |
| I7: Backend ↔ Configuration | Deployment settings → validated app configuration | Environment variables / local call | Missing required settings prevent service readiness. |
| I8: Backend ↔ Logging | Error and operation details → log event | Structured JSON logs | Omit secrets; logging failure must not alter a completed user request. |

### 3.2 API endpoint table

| Method and path | Input | Output | Errors |
| --- | --- | --- | --- |
| GET /api/v1/dishes | Optional query, cuisine | Array of dish summaries | 400, 503 |
| GET /api/v1/dishes/{dishId} | Dish ID | Dish details / rating | 400, 404, 503 |
| GET /api/v1/dishes/{dishId}/reviews | Dish ID | Array of reviews | 400, 404, 503 |
| POST /api/v1/dishes/{dishId}/reviews | Verified token; rating 1–5; comment ≤500 chars | Review ID and updated rating | 400, 401, 403, 404, 409, 503 |
| GET /api/v1/recommendations | Optional preferences/restrictions and limit | Ranked dish array | 400, 401 if authenticated, 503 |
| POST /api/v1/dishes | Firebase ID token, restaurantId, dish name | Dish created and saved in database. Some dish attributes initialized | 400, 401, 404, 409, 503 |
| Photo upload: optional/route pending | JPEG/PNG ≤5 MiB | Photo status / ID | 413, 415, 503 |

Photo upload remains optional under REQ-13.

### 3.3 Data structures and examples

| Method and path | Input | Output | Errors |
| --- | --- | --- | --- |
| GET /api/v1/dishes | Optional query, cuisine | Array of dish summaries | 400, 503 |
| GET /api/v1/dishes/{dishId} | Dish ID | Dish details / rating | 400, 404, 503 |
| GET /api/v1/dishes/{dishId}/reviews | Dish ID | Array of reviews | 400, 404, 503 |
| POST /api/v1/dishes/{dishId}/reviews | Verified token; rating 1–5; comment ≤500 chars | Review ID and updated rating | 400, 401, 403, 404, 409, 503 |
| GET /api/v1/recommendations | Optional preferences/restrictions and limit | Ranked dish array | 400, 401 if authenticated, 503 |
| Dish creation: route pending | Verified user + restaurant ID + dish data | New dish ID | 400, 401, 404, 409 |
| Photo upload: optional/route pending | JPEG/PNG ≤5 MiB | Photo status / ID | 413, 415, 503 |

Example review request:

POST /api/v1/dishes/d42/reviews

Authorization: Bearer <token>

Content-Type: application/json

{

"rating":4,"comment":"Great flavor"

}

Example successful response:

{

"reviewId"

:

"rev7"

,

"dishId"

:

"d42"

,

"rating"

:

4

,

"averageRating"

:

4.5

}

The API must validate all references. A review update changes the existing review and rating totals in one transaction. The backend gets the author ID from the verified token.

## 4. Technology stack

| Layer | Selected choice | Why / related IDs |
| --- | --- | --- |
| Frontend | React | Browser app and mobile- friendly UI (CON-01, REQ- 08, NFR-03). |
| Backend | Python REST API | Existing D1 choice; validation and reviews (REQ- 03, REQ-12). |
| Authentication | Firebase Authentication | Managed login and verified writes (REQ-01, NFR-02). |
| Database | Cloud Firestore | Persistent dish/review records, transactions (CON- 02, NFR-05). |
| Hosting | Firebase Hosting + compatible Python runtime TBD | Web deployment; cost must be checked (CON-02, CON- 04). |
| Recommendations | Internal Python ranking | Low-cost, rule-based suggestions (REQ-02, CON- 04). |
| Photos (optional) | Cloud Storage | Stores dish images outside Firestore (REQ-13). |
| Version control/testing | Git and automated tests | Maintain requirement/test links (CON-05). |

## 5. Known Constraints

| ID | What it forces | What it rules out |
| --- | --- | --- |
| CON-01 | $0 Budget; Free-tier cloud hosting with restricted scaling capabilities | Scaling the platform to accommodate more than what the free tier offers. |
| CON-02 | Data extraction methods must comply with legal regulations on a per website basis | Data from websites with strict terms of service. Any proprietary information. |
| CON-03 | User preference data is prioritized for recommendation algorithm and dietary restrictions | High resolution food photo uploads due to limited free- tier cloud hosting capacity (as discussed in CON-01). |

## 6. Traceability matrix

| Req ID | Description | Design Component (D0/D1/D2) | Status | Test Case |
| --- | --- | --- | --- | --- |
| REQ-01 | Secure sign-in / writes | D2 authentication + D0 API backend | Designed | TC-001 |
| REQ-02 | Recommendations with fallback | D2 ranking module | Designed | TC-002 |
| REQ-03 | Dish reviews and ratings | D1 review schema + D2 backend | Designed | TC-003 |
| REQ-06 | Nearby top-ten dishes | D0 ranking + D2 frontend | Designed | TC-004 |
| REQ-07 | ZIP fallback | D2 frontend | Designed | TC-005 |
| REQ-08 | Dish details | D1 dish + D2 API/frontend | Designed | TC-006 |
| REQ-09 | Search and cuisine filter | D2 API/frontend | Designed | TC-007 |
| REQ-10 | Dietary warnings | D1 profiles + D2 ranking | Designed | TC-008 |
| REQ-11 | Public owner- readable feedback | D2 frontend/API | Designed | TC-009 |
| REQ-12 | Add a dish | D1 Dish + D0 API (route pending) | Designed | TC-010 |
| REQ-13 | Photos (optional) | D1 Photo + D0 storage | Planned | TC-011 |
| REQ-14 | Atomic review updates | D1 Firestore + D2 backend | Designed | TC-012 |
| REQ-15 | Nonexistent dish response | D2 backend + EC-15 | Designed | TC-013 |
| REQ-16 | Unsupported location | D2 frontend + UC-01 | Designed | TC-014 |
| NFR-01 | Search performance | D0 backend + D2 API | Planned | TC-015 |
| NFR-02 | TLS / authorization | D2 Auth + API | Designed | TC-016 |
| NFR-03 | Small-screen usability | D2 frontend | Planned | TC-017 |
| NFR-04 | Staging reliability | D2 backend/error handling | Planned | TC-018 |
| NFR-05 | Rating correctness | D1 transaction + D2 backend | Designed | TC-019 |
| CON-01 | $0 Budget; Free- tier cloud hosting with restricted scaling capabilities | Scaling the platform to accommodate more than what the free tier offers. | Designed | TC-020 |
| CON-02 | Data extraction methods must comply with legal regulations on a per website basis | Data from websites with strict terms of service. Any proprietary information. | Planned | TC-021 |
| CON-03 | User preference data is prioritized for recommendation algorithm and dietary restrictions | High resolution food photo uploads due to limited free-tier cloud hosting capacity (as discussed in CON-01). | Planned | TC-022 |

Legend:

- Designed- anything included in D0/D1/D2. - Planned- anything not fully specified or not yet tested.

Notes:

- Test IDs are proposed for Week 10. - REQ-04 and REQ-05 are both withdrawn.

Coverage:

- 14 active REQs + 5 NFRs + 3 CONs = 22 entries.

## 7. Advisor review

### Status: Pending Advisor Review

Review date: Not yet scheduled/confirmed.

What was discussed: None recorded yet.

Changes resulting from review: None recorded yet.

When the team meets with the advisor, the above section will be updated to reflect that.
