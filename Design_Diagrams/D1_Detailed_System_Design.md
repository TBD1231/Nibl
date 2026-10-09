# **<u>Header, scope, and conventions:</mark></u>** 

Project: Nibl 

Team Members: Jack Keller, Nathan Wiley, Duy Pham, Andrew Ogrin-Cotarlan</mark> 

Goal: Design a system that accommodates the interface, storage, and logic needs of a social app like Nibl. 

Scope: This document details the Database schema, AES algorithm, and API contract.</mark> 

# **<u>Data model, the D1 diagram</mark></u>** 

<img width="562" height="354" alt="image" src="https://github.com/user-attachments/assets/374652ad-1c3c-4916-95aa-0664fbf2614f" />

# **<u>Core algorithms</mark></u>** 

Login password decryption (AES) will be the algorithm that compares the user entered credentials with the encrypted password stored in the database. This solves the problems of security because if they were not encrypted, that is unsecure, as well as login efficiency since you want a login to be quick. The inputs to this algorithm will be the encrypted password text, user inputted plain text, and a decryption key in text. The output will be a Boolean that will determine if the user successfully logs in or not. The expected complexity of this will be O(n) in space and time complexity because as one user is added, one password is added. The time component is because this is the speed of the AES. We chose AES because it adds a good amount of security without compromising speed or memory usage. There should be no edge case because validation and sanitization will be run on inputs, and the algorithm will always have the inputs and keys for it to run. 

# **<u>Build-versus-reuse decisions</mark></u>** 

- Instead of remaking the AES encryption method from scratch, we will use the Python cryptography library which contains the AES encryption methods. 

   - Licensing: Apache-2.0 or BSD-3-Clause</mark> 

   - Age: Made in 2013</mark> 

   - Performance: Positive feedback online for high performance</mark> 

   - Fit: Free, easy to use, high performance, no licensing challenges. It’s a good fit as it poses no challenges or risks and will be easy to set up. 

- There is no template to use to make the database for Nibl specifically, so we will have to design the database from scratch using FireBase built in tools. 

# **<u>API contract</mark></u>** 

- GET</mark> 

   - Purpose: Retrieves a personalized list of recommended food items.</mark> 

   - Inputs:</mark> 

      - user</mark> _ id (string, requied): Unique identifier matching the “Users” entity</mark> 

      - restriction (string, optional): Dietary filter matching “User_Food_Profile.restrictions” 

   - Outputs:</mark> 

      - status (string): Request outcome ("success" or "error").</mark> 

      - dishes (array of objects):</mark> 

         - item_id (string): Unique identifier for the food item matching “Food Items.item_id”. 

         - name (string): Display name matching “Food Items.name”.</mark> 

         - restaurant_name (string): Name from the associated “Restaurants.restaurant_name” entity. 

         - average_rating (float, units: stars out of 5, range: 0.0–5.0): Aggregated score calculated from “Reviews.rating”. 

         - match_score (float, units: relevance score, range: 0.00–1.00): Algorithm match score based on user preferences. 

   - Error Responses:</mark> 

      - 400 Bad Request: Triggered if query parameters have invalid data types.</mark> 

      - 401 Unauthorized: Triggered if an invalid or missing “user_id” authentication token is provided. 

- POST</mark> 

   - Purpose: Submits a new user review and rating for a specific food item.</mark> 

   - Inputs:</mark> 

      - user</mark> _ id (string, required): Foreign key matching the “Users” entity.</mark> 

      - restaurant_id (string, required): Foreign key matching the “Restaurants” entity. 

      - item_id (string, required): Foreign key matching the “Food Items” entity. 

      - rating (integer, required, valid range: 1–5): Numerical rating score matching "Reviews.rating”. 

      - comments (string, optional, max length: 500 characters): Text review comment matching “Reviews.comments”. 

   - Outputs: 

      - review_id (string): System-generated unique identifier for the review. 

      - `o` status (string): Confirmation state ("saved"). 

   - created_at (string): Creation timestamp matching “Reviews.created_at”. 

   - Error Responses: 

      - 400 Bad Request: Triggered if “rating” is outside 1–5, “comments” exceeds 500 characters, or required fields are missing. 

      - 404 Not Found: Triggered if “item_id” or “restaurant_id” does not exist in the database. 

- Example Request and Response: 

JSON

// Example Request Payload

{

  "user_id": "u789",
  
  "restaurant_id": "r12",
  
  "item_id": "item42",
  
  "rating": 4,
  
  "comments": "Great flavor and excellent presentation!"
  
}
 
// Example Response Payload (200 OK)

{
  "review_id": "rev104",
  
  "status": "saved",
  
  "created_at": "2026-10-08T20:30:00Z"
  
} 

- Versioning & Breaking Changes: 

APIs are versioned via the URL path prefix, and a breaking change is defined as removing an existing response field, changing a parameter's data type, or requiring a new mandatory input field. 

# **<u>Technology choices with justification</mark></u>** 

Database/Hosting: FireBase</mark> 

- Selected for strong team skill fit, free hosting cost, active community support, and fast read performance. PostgreSQL and AWS were passed over because paid hosting and schema migration overhead were unnecessary. 

Backend language: Python</mark> 

- Selected for team familiarity, open-source licensing, big community support, zero-cost execution within free limits, and reliable latency for API requests. Node.js was passed over because recommendation logic is written faster in Python. 

Frontend language: React</mark> 

- Selected for team familiarity, permissive licensing, massive community UI ecosystem, fast rendering, and zero-cost hosting on Firebase. Native mobile apps (iOS/Android) were passed over due to Apple hardware testing constraints and higher development costs. 

