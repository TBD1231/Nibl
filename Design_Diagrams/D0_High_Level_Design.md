## Block Diagram (the D0 diagram)

<img width="606" height="498" alt="image" src="https://github.com/user-attachments/assets/442799e8-2262-40f8-b01f-637a0149e410" />

* **Goal:** Design a system that accommodates the interface, storage, and logic needs of a social app like Nibl.
* **System-wide inputs/outputs:** 
  * As this is our standalone system, there are no inputs/outputs to the system.
  * Ultimately, activity starts with a user request and ends with a response to the user.

---

## Component Responsibility Table

| Component | Responsibilities | Input/Output | Primary Owner |
| :--- | :--- | :--- | :--- |
| **App/API Backend** | Querying the database, handling user requests, executing logic. | **Input:** Interface requests, database query results, scored dish recommendations (from recommendation system).<br>**Output:** Processed responses sent back to interface, Database query, requests to recommendation system. | Nathan Wiley |
| **Recommendation System (backend)** | Looking at features like user ratings/similar users and generating dish rankings. | **Input:** Previous ratings/preferences, dish information, ratings from similar users.<br>**Output:** Scored dish recommendations (response to backend). | Jack Keller |
| **Frontend Interface** | Rendering dish discovery feeds in the browser, managing user session state, and capturing review submissions. | **Input:** User actions (searches, ratings), login information, preferences.<br>**Output:** API requests to Backend, rendered web pages, user information. | Duy Pham |
| **Database** | Persists user profiles, restaurant menus, dish ratings in Firestore, stores uploaded photos. | **Input:** Updated data from backend, request from backend.<br>**Output:** Stored data, query results response to backend. | Andrew Ogrin-Cotarlan |

---

## Interface Specification Table

| Connection Input/Output | Format | Protocol | Error Handling | Example Payload |
| :--- | :--- | :--- | :--- | :--- |
| **Frontend to backend**<br>- Sends searches, filters, reviews, ratings, and user preferences<br>*(ID: F1)* | JSON; URL query parameters for searches; multipart form data for photos if we end up using them. | REST over HTTPS, using GET and POST requests. | Backend validates inputs and permissions. Returns an error for invalid data or unauthorized requests. Frontend preserves unsaved input. | `{"dishId":"d42","rating":4,"text":"Great flavor"}` |
| **Backend to frontend**<br>- Returns dish information, recommendations, submission confirmations, or errors.<br>*(ID: B1)* | JSON objects or arrays with HTTP status codes | HTTPS response to the frontend request | Frontend displays error messages, shows an empty state for no results, and allows retry after connection failures. | `{"reviewId":"rev7","status":"saved"}` |
| **Backend to database**<br>- Sends queries and validated records for users, restaurants, dishes, and reviews.<br>*(ID: B2)* | Firestore document fields and query parameters. | Firestore server SDK read/write calls. | Backend validates records, retries temporary failures when safe, and logs persistent errors. Failed writes are not reported as successful. | `{"userId":"u7","dishId":"d42","rating":4}` |
| **Backend to recommendation system**<br>- Sends user preferences, rating history, candidate dishes, and requested result count.<br>*(ID: B3)* | Structured objects and arrays. | Direct internal function call. | Validate required fields. Use popularity-based ranking when user history is unavailable. Return an empty list when there are no candidate dishes. | `{"preferences":["spicy"],"history":[],"candidates":[{"dishId":"d42","tags":["spicy"],"averageRating":4.5}],"limit":20}` |
| **Database to backend**<br>- Returns stored records, query results, write acknowledgments, or database errors.<br>*(ID: D1)* | Firestore document/query snapshots and write results. | Result of the corresponding Firestore SDK call. | Backend distinguishes empty results from failures, handles missing records, and converts database errors into appropriate API responses. | `{"dishId":"d42","averageRating":4.5,"tags":["spicy"]}` |
| **Recommendation system to backend**<br>- Returns ranked dish IDs and recommendation scores<br>*(ID: R1)* | Ordered array of objects containing dish IDs and numeric scores | Return value from the internal function call | Backend validates returned IDs and scores. Uses a nonpersonalized ranking if recommendation processing fails. | `[{"dishId":"d42","score":0.92}]` |

---

## Data-Flow Diagram

<img width="594" height="478" alt="image" src="https://github.com/user-attachments/assets/4ae50cbd-a35a-4e84-a63a-793caab4270f" />

## Architecture Pattern and Justification

### Layered / 3-tier
* The interface and backend communicate, the backend and the database communicate, and the recommendation system and backend communicate. The interface, database, and recommendation systems do not directly communicate.
* This pattern governs the entire Nibl system.

#### Justification:
* **Fit to the problem:** It is a natural fit for a web application because there are 3 clear needs of a frontend, backend, and storage layer. In the frontend, users see restaurants and dishes in a human readable form. In the backend, input is validated, frontend requests are processed, and actions are taken automatically on the users' behalf. The data tier keeps certain data persistent across sessions, like users’ login and preferences, ratings, menu items, restaurants, etc.
* **Team skills:** Several members have built websites in the past using a layered architecture.
* **Performance and Timing:** Nibl doesn't have extremely strict real-time requirements. Normal web-request latency is sufficient for searches, reviews, ratings, and recommendations.
* **Scalability:** For the expected user volume and needs (piloting in the Cincinnati area), this architecture should easily meet user needs. If necessary, the selected hosting service is easily scalable with automatic resource expansion.
* **Hardware Constraints:** Nibl is a cloud-based web application. The presentation tier runs on the user's device/browser, while application logic and data storage can run in cloud infrastructure such as Firebase.

> We rejected a microservice pattern as an alternative because it adds unnecessary complexity. The application is not large enough to justify having different services that need to be developed and run independently.

---

## Decision Log

| Decision | Alternatives | Reasoning |
| :--- | :--- | :--- |
| **We will develop algorithms for a user's recommendations** | AI APIs, Local LLM, Machine Learning | Using any of the alternatives will add additional monetary cost, reduce the response time, and add unnecessary system complexity. |
| **We will use Firebase as an all-in-one hosting solution.** | AWS, Supabase | Some team members have experience with firebase. It is free but scalable for a cost. Firebase also provides hosting for the storage, web interface, and backend components, so it is an all-in-one solution. |
| **We will be developing a website for this project.** | Android or IOS app | We decided to create a website instead of an app because the whole team has an apple phone, but not an apple computer. This would make an android app hard to test, and an IOS app would incur more cost for development and hosting. |
| **We decided to split the backend into two subsystems.** | Monolithic backend | Splitting up the backend allows us to work separately without impacting the other's performance, as well as reducing complexity within a single component. |
