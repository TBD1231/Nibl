# Restaurant Dish Review Application - Andrew Ogrin-Cotarlan, 

## Economic
Our strict $0 budget restricts our backend to free-tier cloud hosting services, directly capping our database capacity and monthly bandwidth limit. Additionally, our explicit time budget of 225 hours across four team members over two semesters forces us to prioritize core dish-review functionalities and defer secondary application features. 

## Legal
Because our application aggregates menu item data from local restaurants, we must strictly comply with individual website Terms of Service regarding automated data extraction. This legal constraint shapes our solution by requiring our data-gathering tools to only target publicly available, factual menu prices and dish names rather than proprietary descriptions. Consequently, our database schema is explicitly designed to store only standardized dish metadata and user-generated text to entirely avoid intellectual property disputes.

## Social
Our project aims to provide a unique societal benefit by shifting the dining focus from general restaurant atmosphere reviews to highly specific, itemized dish ratings. To meaningfully improve the user’s experience, our search algorithm must be engineered to filter by granular dietary restrictions and specific user tastes. This design choice ensures our application serves diners with strict allergies, allowing them to find safe, highly-rated meals with confidence. We face a concrete trade-off between our zero-dollar economic constraint and our social goal of providing a visually comprehensive user experience. Because free database tiers severely limit our storage capacity, we balanced this tension by restricting users to text-only dish reviews, deliberately dropping high-resolution food photo uploads to conserve server space.
