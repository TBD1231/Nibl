# Team Members
* Jack Keller (CS) - kelle3jk@mail.uc.edu
* Andrew Ogrin-Cotarlan (CS) - ogrincaw@mail.uc.edu
* Nathan Wiley (CS) - wileynn@mail.uc.edu
* Duy Pham (CS) - phamdd@mail.uc.edu

# Stakeholder Map

| Primary | Secondary | Hidden |
| :--- | :--- | :--- |
|Restaurant goers looking for dish recommendations<br>Restaurant goers looking to leave a dish review | Restaurant Owners | People with Dietary restrictions |

# User Stories with Acceptance Criteria

**US-01 (Primary):** As a Food Connoisseur, I want to leave reviews and have recommendations for similar new restaurants and foods, so that I can find new foods and restaurants, as well as enjoy my experience more.
* **AC-01.1:** Given that I want to try new foods, When I am on the Nibl website, Then when I look at a restaurant, I can see meals with ratings.

**US-02 (Secondary):** As a Restaurant Owner, I want to have my restaurant gain more popularity through our great menu items, so that I can consistently fill up my restaurant with customers.
* **AC-02.1:** Given I'm not sure why one of my menu items is not doing well, When I read negative reviews pinpointing the cause of this, Then I change the recipe of the menu item according to the reviews or just scrap the item.

**US-03 (Hidden):** As a Person with a dietary restriction, I want to be informed if a meal violates any of my dietary restrictions, so that I don't accidentally eat something hazardous to my health/religion/beliefs.
* **AC-03.1:** Given that I have dietary restrictions, When I look for a dish to order, Then I should be able to see visual indicators that the dish violates the restriction.

# Use Cases

**UC-01: Discover Top Dishes at a Nearby Restaurant**
* **Expands Story:** US-01
* **Use Case Name:** Discover Top Dishes
* **Primary Actor:** Cincinnati Local Diner
* **Secondary Actors:** n/a
* **Preconditions:** 
  * The user has granted the application permission to access the service location.
  * The database contains verified menu items for restaurants in the user's vicinity.
* **Main Success Flow:**
  1. **Actor:** Requests top-rated dishes within a 5-mile radius of their current location
  2. **System:** Queries the database for restaurants matching their location and ranks their individual menu items by user rating.
  3. **System:** Returns a list of the top 10 individual dishes, displaying dish name, restaurant name, distance, and rating.
  4. **Actor:** Selects a specific dish from the list to view detailed feedback.
  5. **System:** Displays the dish detail profile, rating breakdowns, user photos, recent reviews.
* **Alternate Flow (AF-1: Filtering by Cuisine):**
  1. At Step 3 of Main Flow, the actor chooses to filter the results by a specific cuisine.
  2. System filters the active nearby dish list to display only dishes categorized under the chosen cuisine.
  3. System updates the display to show the top-rated matching dishes.
  4. Workflow rejoins Main Flow at Step 4.
* **Exception Flow:**
  1. At step 3 of Main flow, the system determines that there isn't enough data to return to the user because their location is outside of the supported Cincinnati area. An error message is provided.
* **Postcondition:** The actor views dish-specific rating data for a selected item near their target location.

# Given / When / Then Acceptance Criteria

**Acceptance Criteria for UC-01 (Discover Top Dishes)**

**AC-1.1 (Main Success Flow):**
* **Given:** The user has location services enabled.
* **When:** The user executes a request for top nearby dishes.
* **Then:** The system displays a list of up to 10 dishes sorted by rating in descending order, where 100% of listed items are within a 5-mile radius and have an average score derived from at least 1 verified user rating.

**AC-1.2 (Exception Flow - Location Disabled):**
* **Given:** Location services are disabled on the user's device.
* **When:** The user requests nearby dishes.
* **Then:** The system displays a location prompt, accepts a zip code entry, and returns results centered on that zip code.
