# 1. D1 Overview

## Data model, the D1 diagram

Project: Nibl

Goal: Design a system that accommodates the interface, storage, and logic needs of a social app like Nibl.

Legend: Boxes are entities, arrows represent relationships between entities

<img width="717" height="433" alt="image" src="https://github.com/user-attachments/assets/721f9fe3-0aa5-4021-bb92-5b7ceb715f2b" />

*Figure 0: Data Model*

### Summary of D1

- What is Nibl? Nibl is a social platform for helping people discover their favorite dishes at their favorite Cincinnati restaurants. Nibl focuses on dishes rather than restaurants, featuring a recommendation algorithm that factors in dietary restrictions, user preferences, and user data (don't like spicy food, looking at previous ratings, similar users, etc.) to personally rank dishes for users. Users can write or view reviews for dishes at supported restaurants. Nibl is being implemented as a website, using Firebase as an all-in-one hosting solution. We are using a layered architecture (frontend, backend, database).

- What happened in D1? D1 covered the database component of Nibl. The rough database schema can be seen in the above Figure 0. We settled on how the passwords will be stored in the database, based on the encryption algorithm selected. We designed an API contract that related entities and their attributes to API calls like GET and POST.

- How does D2 expand on D1? D2 details the design plan for the frontend, backend, and recommendation system components of Nibl that were absent in D1. In addition, D2 examines edge cases and error handling, requirements traceability, and design choices with rationale.

- What's missing or being questioned? Currently, we aren't sure how we are going to have data when we launch the app (i.e. on launch when there are no users, new users may come in and see there are no reviews and have a bad experience). The database schema is also being questioned, as we are unsure of the best way to go about organizing the relationships between dietary restrictions, food preferences, ingredients, and meals in a way that considers the needs of the user.

# 2. D2

## 1) Frontend

### Data model

- No persistent data

- Retrieves app data from the backend. Any state here is temporary and is lost based on browser behavior.

### Algorithm/Logic

#### Dish Search and Review Submission

1. The user opens Nibl and searches for a dish or restaurant.

1. The frontend validates the search input and sends a request to the backend.

1. The backend returns matching dishes, including their names, restaurants, and average ratings.

1. The frontend displays the results and allows the user to select a dish.

1. When a dish is selected, the frontend requests its details and existing reviews.

1. An authenticated user can submit a rating from 1 to 5 and an optional written review.

1. The frontend validates the rating and submits the review to the backend.

1. If successful, the frontend refreshes the dish's rating and reviews.

1. If unsuccessful, an error message is displayed without discarding the user's input.

**Edge case: Empty searches return an empty result set or default results; failed requests display an error; unauthenticated users are prompted to sign in before submitting reviews.**

### Interface/Endpoints

The frontend communicates with the backend using HTTPS requests and JSON responses.

| Method | Endpoint | Purpose |
| --- | --- | --- |
| GET | /api/v1/dishes?query={string} | Search for dishes |
| GET | /api/v1/dishes/{dishId} | Retrieve dish details |
| GET | /api/v1/dishes/{dishId}/reviews | Retrieve reviews |
| POST | /api/v1/dishes/{dishId}/reviews | Submit a review |
| GET | /api/v1/recommendations | Retrieve personalized recommendations |

#### Example JSON Payload/Response

```json
// Example Request Payload
{
"user_id": "u789",
"restaurant_id": "r12",
"item_id": "item42",
"rating": 4,
"comments": "Great flavor and excellent presentation!"
}
```

```json
// Example Response Payload (200 OK)
{
"review_id": "rev104",
"status": "saved",
"created_at": "2026-10-08T20:30:00Z"
}
```

#### Error Responses

- 400 Bad Request: Triggered if “rating” is outside 1–5, “comments” exceeds 500 characters, or required fields are missing.

- 404 Not Found: Triggered if “item_id” or “restaurant_id” does not exist in the database.

## 2) Backend

### Data model

- No persistent data

- Uses the database component from D1 for persistent data.

### Algorithm/Logic

#### Review Processing and Rating Update

1. The backend receives a review submission from the frontend.

1. It verifies the user's Firebase authentication token.

1. It validates the dish identifier, rating, and review text.

1. It checks that the dish exists in the database.

1. It checks whether the user has already reviewed the dish.

1. If a review exists, the backend updates it; otherwise, it creates a new review.

1. The backend updates the dish's aggregate rating information using a database transaction to prevent inconsistent totals.

1. The updated review and dish rating are returned to the frontend.

**Edge case: Invalid ratings are rejected; nonexistent dishes return an error; duplicate submissions update an existing review rather than creating multiple reviews. Database failures are reported without returning a false success response.**

### Interface/Endpoints

The backend exposes REST API endpoints to the frontend and internal methods to the database and recommendation system.

| Method | Endpoint | Input | Output |
| --- | --- | --- | --- |
| GET | /api/v1/dishes | Optional query: string | DishSummary[] |
| GET | /api/v1/dishes/:dishId | Required dishId: string | DishDetails |
| GET | /api/v1/dishes/:dishId/reviews | Required dishId: string | Review[] |
| POST | /api/v1/dishes/:dishId/reviews | rating: integer, comment?: string | Created or updated Review |
| GET | /api/v1/recommendations | Optional limit: integer | Recommendation[] |

#### Example HTTP Request

```http
POST /api/v1/dishes/dish123/reviews
Authorization: Bearer <firebase-id-token>
Content-Type: application/json
```

```json
{
  "rating": 5,
  "comment": "Excellent flavor and portion size."
}
```

#### Example JSON Response

```json
{
  "reviewId": "review789",
  "dishId": "dish123",
  "userId": "user456",
  "rating": 5,
  "comment": "Excellent flavor and portion size.",
  "averageRating": 4.6
}
```

#### Error Responses

| Code | Condition |
| --- | --- |
| 400 | Missing or invalid authentication token |
| 401 | Missing or invalid authentication token |
| 403 | User attempts an unauthorized modification |
| 404 | Dish or review does not exist |
| 409 | Conflicting concurrent modification |
| 500 | Unexpected backend or database failure |
| 503 | Required service temporarily unavailable |

## 3) Recommendation System

### Data model

- No persistent data

- Retrieves necessary data from the backend (which the backend gets it from the database).

### Algorithm/Logic

#### Personalized Dish Ranking

1. Retrieve the user's preferences and rating history.

1. Retrieve candidate dishes from the database.

1. Exclude dishes containing known allergens or violating the user's dietary restrictions.

1. Exclude dishes the user has already reviewed.

1. Calculate a recommendation score for each remaining dish using its average rating, cuisine preference match, and similarity to previously liked dishes.

1. Sort candidates by recommendation score in descending order.

1. Return the highest-ranked dishes up to the requested limit.

**Edge case: New users without rating histories receive recommendations based on preferences and popular dishes. If no dishes satisfy the filters, an empty array is returned. Tied scores are resolved using average rating, followed by dish ID for deterministic ordering. Dishes with unknown allergen information are excluded when the system cannot verify that they satisfy a user's allergy restrictions.**

### Interface/Endpoints

The recommendation system communicates with the backend through an internal function rather than exposing its own public REST endpoint.

#### Function Signature

```typescript
generateRecommendations(
  request: RecommendationRequest
): Promise<Recommendation[]>
```

#### Request Signature

```typescript
interface RecommendationRequest {
  userId: string;
  limit: number; // Integer from 1 to 50
}
```

#### Response Structure

```typescript
interface Recommendation {
  dishId: string;
  restaurantId: string;
  name: string;
  score: number; // Range: 0.0 to 1.0
  averageRating: number; // Range: 0.0 to 5.0
}
```

#### Example JSON Output

```json
[
  {
    "dishId": "dish123",
    "restaurantId": "rest456",
    "name": "Grilled Salmon",
    "score": 0.92,
    "averageRating": 4.7
  },
  {
    "dishId": "dish234",
    "restaurantId": "rest567",
    "name": "Vegetable Curry",
    "score": 0.86,
    "averageRating": 4.5
  }
]
```

**Errors: INVALID_REQUEST for an invalid limit or user ID, USER_NOT_FOUND for a nonexistent user, and DATA_UNAVAILABLE when required database information cannot be retrieved. No matching dishes is a successful empty result, not an error.**

# 3. Edge Cases and Error Handling

| ID | Component | Category | Edge case | Design decision |
| --- | --- | --- | --- | --- |
| EC-01 | Frontend | Boundary values | Search returns no dishes. | Frontend: Display “No dishes found.” Recoverable by changing the search. |
| EC-02 | Frontend | Timing/concurrency | Search responses arrive out of order. | Frontend: Ignore older responses and display the latest search. Automatically recoverable. |
| EC-03 | Frontend | Bad/missing input | Rating is missing or invalid. | Frontend: Block submission and show a validation message. Recoverable by correcting the rating. |
| EC-04 | Frontend | External dependencies | Backend request fails. | Frontend: Keep entered information, show an error, and offer Retry. Recoverable when connection returns. |
| EC-05 | Backend | Boundary values | Rating is outside 1–5 or comment exceeds 500 characters. | Backend validator: Return 400 without saving. Recoverable by correcting input. |
| EC-06 | Backend | Timing/concurrency | Duplicate review submissions arrive together. | Backend review handler: Update one user/dish review atomically; return 409 if conflicts persist. Recoverable by reloading and retrying. |
| EC-07 | Backend | Bad/missing input | Authentication token is missing or invalid. | Backend authentication handler: Return 401 without saving. Recoverable by signing in again. |
| EC-08 | Backend | External dependencies | Firebase authentication is unavailable. | Backend authentication handler: Return 503; do not allow unauthenticated writes. Recoverable when service returns. |
| EC-09 | Recommendations | Boundary values | New user has no rating history. | Ranking handler: Fall back to preferences and popular dishes while keeping dietary filters. Recoverable as history grows. |
| EC-10 | Recommendations | Timing/concurrency | Preferences change during ranking. | Frontend recommendation handler: Discard the old response and request updated recommendations. Recoverable automatically. |
| EC-11 | Recommendations | Bad/missing input | Allergen information is missing for a restricted user. | Ranking handler: Exclude that dish; return an empty list if none qualify. Recoverable when reliable data is added. |
| EC-12 | Recommendations | External dependencies | Required recommendation data cannot be retrieved. | Recommendation handler: Return DATA_UNAVAILABLE; backend returns 503. Recoverable when data access returns. |
| EC-13 | Database | Boundary values | Dish has zero reviews. | Database rating handler: Avoid division by zero; return no average and display “Unrated.” Recoverable after the first review. |
| EC-14 | Database | Timing/concurrency | Reviews update the same dish simultaneously. | Database transaction handler: Save reviews and rating totals atomically; retry up to three attempts. Recoverable; return conflict if exhausted. |
| EC-15 | Database | Bad/missing input | Review references a nonexistent dish. | Database repository: Reject the write; backend returns 404. Recoverable by selecting an existing dish. |
| EC-16 | Database | External dependencies | Database is unreachable. | Database adapter: Return an unavailable error; backend returns 503 without claiming success. Recoverable when service returns. |

# 4. Requirements Traceability Table

| Requirement ID | Type | Description | Priority |
| --- | --- | --- | --- |
| REQ-01 | Functional | Users must be able to log in securely. | High |
| REQ-02 | Functional | Users must receive personalized dish recommendations based on dietary profiles. | High |
| REQ-03 | Functional | Users must be able to rate and review individual food items. | High |
| REQ-04 | Non-Functional | System must operate free-of-charge for basic hosting tier. | Medium |
| REQ-05 | Non-Functional | Mobile client user interface must render quickly. | Medium |

# 5. Design Rationale

Database/Hosting: FireBase

Selected for strong team skill fit, free hosting cost, active community support, and fast read performance. PostgreSQL and AWS were passed over because paid hosting and schema migration overhead were unnecessary.

Backend language: Python

Selected for team familiarity, open-source licensing, big community support, zero-cost execution within free limits, and reliable latency for API requests. Node.js was passed over because recommendation logic is written faster in Python.

Frontend language: React

Selected for team familiarity, permissive licensing, massive community UI ecosystem, fast rendering, and zero-cost hosting on Firebase. Native mobile apps (iOS/Android) were passed over due to Apple hardware testing constraints and higher development costs.

Testing of AI usage versus Algorithms

In designing the recommendation system, we will do multiple tests on different data sets to compare if we should use AI or straight algorithms for what is recommended for a specific user. The leading case is using algorithms because of the financial costs associated with AI.

Locations: Cincinnati

We have decided to constrain our data sets to locations in Cincinnati. This is because of the financial costs associated with storing information beyond this within our database. We will try to maximize the radius of restaurants according to our constraints

Solution Type: Website

We decided to host a website rather than develop an Android or IOS app because of the additional costs of developing apple applications, and we had no capacity to test Android apps within our team.

# 6. Peer Design Review Notes

| ID | Criteria | Issue | Raised By | Action | Owner | Priority | Due Date |
| --- | --- | --- | --- | --- | --- | --- | --- |
| PR-1 | Feasibility | Login algorithm security | Group 11 GE | Investigate alternative login algorithms that offer enhanced security | Nathan | Must-fix | 10/20 |
| PR-2 | Clarity | Endpoint parameters example | Group 11 GE | Add data example to endpoint parameters | Luke | Worth considering | 10/22 |
| PR-3 | Clarity/Completeness | Adding general synopsis of app to documentation | Group 15 Fish-your price | Review documentation and add a summary if needed | Andrew | Not necessary | 10/27 |
| PR-4 | Clarity | How users register allergens into their preferences | Group 15 Fish-your price | Review frontend component documentation and add description of process | Jack | Worth considering | 10/22 |
