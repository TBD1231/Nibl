# Nibl - Ethics and Responsible-Use Statement

**Date:** November 11, 2026  
**Team:** Andrew Ogrin-Cotarlan, Nathan Wiley, Duy Pham, Jack Keller

## 1. Privacy summary

This table covers the D0–D2 data flows and the Week 9 technical specification document. The retention rules are proposed and need team approval.

**A:** Until account deletion. **C:** Until corrected, removed, or the pilot ends. **T:** Current request or session. **L:** Until logs expire. **V:** Provider retention. **F:** Deferred from the pilot.

| Data item | Source | Purpose | Consent / notice | Storage | Access | Retention |
|---|---|---|---|---|---|---|
| Sign-in email, if used | Diner | Identify sign-in account | Signup notice before submission | Firebase Auth | Diner, Auth provider, authorized admins | A/V |
| Password, if used | Diner | Authenticate | Submit to Firebase sign-in | Firebase Auth handles credential | Diner and Firebase Auth | A/V |
| ID token | Firebase Auth | Authenticate API calls | Sign-in notice | Firebase SDK session, API request | Browser, verifier | T/V |
| Refresh token | Firebase Auth | Renew session | Sign-in notice | Firebase SDK session | Browser SDK and Auth | V |
| User ID / UID | Auth verifier | Ownership, join records | Signup notice | Auth, Firestore | API/admin, proposed private | A |
| Username | Diner, D1 Users | Display identity if enabled | Explain visibility before saving | Firestore | Public only if disclosed | A |
| Account created_at | Server, D1 Users | Account record timestamp | Signup notice | Firestore | API/admin | A |
| Dietary restrictions | Diner | Flag/filter conflicts | Optional opt-in before saving | Firestore profile | Diner and authorized API/admin | A or clear |
| Food preferences | Diner | Rank dishes | Optional opt-in before saving | Firestore profile | Diner and authorized API/admin | A or clear |
| Profile updated_at | Server | Track profile revision | Profile notice | Firestore | Diner and authorized API/admin | A |
| Rating history | Reviews via API | Personalize ranking | Explain reuse before opt-in | Derived from Firestore reviews | API/ranking | T, source A |
| Restaurant ID | Seed/admin | Join dishes to restaurant | Seed provenance review | Firestore | Public via API | C |
| Restaurant name | Seed/admin | Identify restaurant | Seed provenance review | Firestore | Public via API | C |
| Restaurant address | Seed/admin | Show restaurant location | Seed provenance review | Firestore | Public via API | C |
| Restaurant coordinates | Seed/admin | Compute nearby distance | Seed provenance review | Firestore/fixture | API, public if exposed | C |
| Dish ID (item_id) | Server/seed | Identify dish across entities | Publish notice on creation | Firestore | Public via API | C |
| Dish name | Diner/seed | Search and display | Publish notice on creation | Firestore | Public via API | C |
| Dish restaurant_id | Diner selects existing ID | Link dish to restaurant | Publish notice on creation | Firestore | Public via API | C |
| Cuisine category | Seed/catalog | Filter and rank | Seed provenance review | Firestore | Public via API | C |
| Ingredient name | Seed/catalog | Describe known ingredients | Seed provenance review | Firestore | Public via API | C |
| Ingredient item_id | Seed/catalog | Link ingredient to dish | Seed provenance review | Firestore | API | C |
| Dietary tags / reported status | Seed/catalog | Show conflict, reported absent, unknown | Seed provenance review | Firestore | Public via API | C |
| Review ID | Server | Identify review | Publish notice on posting | Firestore | Public via API | A |
| Review author UID | Verified token | Enforce review ownership | Explain account linkage | Firestore | API/admin, proposed private | A |
| Review dish ID | Route / server | Link review to dish | Publish notice on posting | Firestore | Public via API | A |
| Review restaurant_id | Dish link in D1 | Associate review with venue if retained | Publish notice on posting | Firestore | Public via API if exposed | A |
| Star rating | Diner | Public rating and totals | Explain visibility before posting | Firestore | Public via API | A |
| Review comment | Diner, optional | Public item feedback | Explain visibility before posting | Firestore | Public via API | A |
| Review created_at | Server | Date/order reviews | Publish notice on posting | Firestore | API, public if displayed | A |
| Rating sum | Review transaction | Calculate average | Explain rating aggregation | Firestore | API/admin | C, recompute on removal |
| Rating count | Review transaction | Display volume and calculate average | Explain rating aggregation | Firestore | Public via API | C, recompute on removal |
| Average rating | Sum / count | Display and rank | Explain rating aggregation | API / Firestore totals | Public via API | C, recompute on removal |
| Search query | Visitor | Match dish or restaurant | Search form notice | Browser/API request | Browser/API | T |
| Cuisine filter | Visitor | Narrow matches | Explicit filter selection | Browser/API request | Browser/API | T |
| Latitude | Browser geolocation | Nearby ranking | Explain purpose, then browser permission | Browser/API request | Browser/API | T |
| Longitude | Browser geolocation | Nearby ranking | Explain purpose, then browser permission | Browser/API request | Browser/API | T |
| ZIP | Visitor | Nearby lookup if location declined | Explain fallback on entry | Browser/API request | Browser/API | T |
| ZIP centroid | Configured lookup | Resolve ZIP to search origin | Disclose approximate result | Lookup/fixture | API | C |
| Search radius | Service default (5 mi) | Bound nearby candidates | Discovery UI notice | Config/request | API | T for request |
| Result limit | Client/API | Bound response | Feature notice | Request | API/ranking | T |
| Recommendation score | Internal ranking | Order candidates | Explain optional personalization | API memory / response if emitted | Ranking/API, requesting diner | T |
| Unsaved review draft | Browser | Retry failed submission | Explain when saving fails | Browser state | Diner | T |
| Operation / route template | API or host | Diagnose requests | Operational logging notice | Restricted logs | Operators | L/V |
| Status / error code | API or host | Identify failures | Operational logging notice | Response and restricted logs | Requester for response, operators for logs | T/L |
| Log timestamp | API or host | Correlate events | Operational logging notice | Restricted logs | Operators | L/V |
| Network IP | Hosting edge | Request delivery / possible abuse controls | Disclose provider handling | Edge, verify any provider logs | Provider / authorized operator | V |
| Deployment settings | Runtime | Start and configure API | Operator-managed | Environment | Runtime/operator | Until superseded |
| Service credentials | Runtime identity or secret store | Authorize server SDK | Operator-managed | Managed environment | Runtime/operator | Until rotated or retired |
| Image bytes | Diner, JPEG/PNG ≤5 MiB | Optional dish photo | Future explicit upload notice | Cloud Storage if enabled | Future policy review | F |
| Photo ID | Server | Track photo | Future upload notice | Firestore if enabled | API | F |
| Object key | Storage/API | Locate or delete upload | Future upload notice | Firestore / Cloud Storage | API/admin | F |
| Photo-to-dish link | Upload target | Associate image with dish | Future upload notice | Firestore if enabled | API | F |

**Consent.** At signup, explain what Firebase collects. Ask separately before saving optional dietary preferences or using browser location. Offer ZIP entry if location is declined. Tell diners before posting that their ratings and comments are public.

**Deletion plan.** Revoke sessions, delete the user’s profile and reviews, update the affected dish totals in Firestore, then delete the Firebase Auth account. At the end of the pilot, disable submissions and delete pilot records, accounts, exports and any uploads.

**Removed field.** The Week 9 design drops the app’s encrypted password field and decryption key from D1. Firebase handles passwords. This change has been confirmed by the team.

## 2. Security summary

**Default credentials.** We will keep demo accounts and seeded test users in the Firebase emulators. Each team member who needs admin access will use their own account. Before launch we’ll make sure that the live Firebase project doesn’t have these demo or test accounts.

**Unauthenticated endpoints.** Visitors can search for dishes, view dish details, and read public reviews. Adding a dish or posting or editing a review requires a Firebase ID token. The Python API will verify the token with the Firebase Admin SDK and check that the signed-in user owns the review before allowing an edit. ID tokens should last about an hour. We’ll check for revoked sessions on writes and require HTTPS with TLS 1.2 or later.

**Secrets in source control.** We’ll use a managed service identity if our hosting platform supports it and keep any other credentials in the host’s secret store. We’ll keep service-account keys and local `.env` files out of Git and leave tokens and other important values out of logs. We will also make sure none of these important values are accidentally committed and will do so with a Git history scan.

**Rate limiting.** For staging, we’ll start with 120 public read requests per minute per IP address and 10 write requests per minute per signed-in user. We’ll check that normal browsing works and that requests over the limit receive HTTP 429, then adjust the limits if needed.

| Self-audit question | Answer |
|---|---|
| Are there default credentials? | We don’t plan to use shared admin credentials. Team members who need admin access will use individual accounts. Test accounts will stay in the Firebase emulators. |
| Which endpoints work without login? | Visitors will be able to search for dishes, view dish details, and read public reviews. Adding a dish or posting a review will require signing in. Users will only be able to edit their own reviews. |
| How long are tokens valid, and is traffic encrypted? | Firebase ID tokens last about one hour. Nibl will require HTTPS with TLS 1.2 or later. In staging, we’ll test that a revoked session can’t submit a write. |
| Where do secrets live? | We plan to use a managed service identity if our host supports it. Any credentials the backend still needs will go in the host’s secret store. We’ll keep credentials and tokens out of Git and application logs. |
| What did the Git history scan find? | We haven’t started implementation or run the scan yet. Once we start committing code and configuration, we’ll scan the full Git history and record the results. |
| Is rate limiting enabled? | Rate limiting is planned. In staging, we’ll start with 120 public reads per minute per IP address and 10 writes per minute per signed-in user. We’ll test that normal browsing works and that requests over the limit return HTTP 429. |

**Ethics and constraints.** To reduce the risk of misleading users who have allergies, Nibl will label missing allergen information as “unknown” and leave those dishes out of personalized results for users who have these allergies. Users will be able to decline location access and enter a general ZIP code instead, and their dietary preferences will stay private. The site will support mobile browsing as well so users can access it without a desktop. We’ll check permission for third-party menu data before using it and credit the source where required.

## 3. Licensing summary

The table below lists the tools and dependencies planned in D1 and Week 9. Since we haven’t started implementation, package versions are still to be determined. Once we set up the project, we’ll record the versions we install and add any other packages, datasets, or assets we use.

| Dependency, dataset or asset | Version | SPDX / terms | Type | Obligation or disposition |
|---|---|---|---|---|
| React | TBD | MIT | Permissive | Keep the copyright and license notice in distributed copies. |
| Python runtime | TBD | PSF-2.0 | Permissive | Keep the applicable notices if we distribute Python with Nibl. |
| Firebase JavaScript SDK | TBD | Apache-2.0 | Permissive | Include the license and applicable notices if we distribute the SDK. Mark any files we modify. |
| Firebase Admin Python SDK | TBD | Apache-2.0 | Permissive | Include the license and applicable notices if we distribute the SDK. Mark any files we modify. |
| Google Cloud Firestore Python client | TBD | Apache-2.0 | Permissive | Include the license and applicable notices if we distribute the client. Mark any files we modify. |
| Python cryptography (D1 AES proposal) | None selected | Apache-2.0 OR BSD-3-Clause | Permissive | Considered for the original AES design. Once we set up the project, check whether another package installs it as a dependency. |
| Firebase Auth, Hosting and Firestore | Managed service | Firebase service terms | Service terms | Follow the applicable service terms when setting up and using Firebase. |
| Cloud Storage | Deferred | Provider service terms if enabled | Service terms | If we add photo uploads, check the service terms and decide how to handle photo permissions and removal requests. |
| Week 10 test data (D01–D12, U1/U2 and sample reviews) | Week 10, Section 5 | No SPDX license assigned | Team-created test data | Use made-up data and label it as test data. |
| Live menu data, fonts, icons, images and code snippets | None selected yet | TBD for each item | TBD | Record the source, license and any required credit for each item we use. |

**Project license:** We haven’t chosen a license for Nibl yet. The planned software dependencies listed above use permissive licenses. Once we choose a project license and finalize our dependencies, we’ll check compatibility and include any required license notices.

## 4. Attribution list

**Code:** Our planned stack includes React (MIT), Python (Python Software Foundation License), and the Firebase JavaScript SDK, Firebase Admin Python SDK, and Google Cloud Firestore Python client (Apache-2.0). We considered PyCA cryptography for the original AES design in D1.

**Data:** Our Week 10 test plan uses team-created sample restaurants, dishes, reviews, and locations. We haven’t selected any live menu data sources yet. If we use live menu data, we’ll record where it came from, how we collected it, and any restrictions on its use.

**Snippets and assets:** Implementation hasn’t started, so we have no adapted code snippets or third-party fonts, icons, or images to credit yet. If we use any, we’ll record the author, source, and license in the project README or NOTICE file.

## 5. AI-use disclosure

> **Syllabus Policy:** "Generative AI tools may be used for drafting and code assistance but not for the design decisions or written analysis that are assessed in this course."

| Tool | Used by | Task | Column | How verified |
|---|---|---|---|---|
| ChatGPT | Andrew Ogrin-Cotarlan | Helped draft, simplify, and format the Week 9–11 documents after the team agreed on the main ideas. | Generally OK | Compared the edits with the original material to make sure the meaning stayed the same. |
| ChatGPT | Jack Keller | Converted the Week 9 and 10 Word documents to Markdown for the repo. | Generally OK | Compared the Markdown with the original Word documents line by line. |
| ChatGPT | Nathan Wiley | Helped draft and simplify wording after the team agreed on the main ideas. | Generally OK | Compared the rewrite with the original draft to make sure the meaning stayed the same. |
| ChatGPT | Duy Pham | Helped draft and simplify wording after the team agreed on the main ideas. | Generally OK | Compared the rewrite with the original draft to make sure the meaning stayed the same. |

**Team confirmation.** By signing below, each member confirms that the architecture, data model, priorities and design rationale were the team’s own.

- Andrew Ogrin-Cotarlan — Signature: Andrew Ogrin-Cotarlan, Date: 11/11/2026
- Nathan Wiley — Signature: Nathan Wiley, Date: 11/11/2026
- Duy Pham — Signature: Duy Pham, Date: 11/11/2026
- Jack Keller — Signature: Jack Keller, Date: 11/11/2026
