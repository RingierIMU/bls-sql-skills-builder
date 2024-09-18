# MySQL Skill Builder Challenge. 

---

**Welcome to the MySQL Skill Builder Challenge!**

Today’s session is designed to give you a chance to flex some mysql muscle. 
There are some basic, and some intermediate questions in this challenge. 
But no matter your level of exposure or experience with MySql, all questions could be answered with a little help from your AI friends.

All of these questions represent the types of queries we might be expected to write in our day-to-day work as developers, or questions we might ask in our roles as non developers.

As part of this process, we encourage you to use tools like Google and AI where appropriate. 
These technologies are powerful force multipliers in our work, and understanding how and when to leverage them effectively is key. 

In helping to track the impact of these tools, we ask for feedback on each question to indicate which tools (if any), you decided to lean on. If you don't need them, don't use them. But treat it as you would for a normal daily task, reaching out where required for the extra assistance. 

You may wonder, why focus on MySQL? While MySQL is nothing new, and often abstracted away by ORMs, 
it remains an essential tool in our broader skill set. Even though many of our staff don’t use it directly every day, fluency in SQL is fundamental across various roles—from QAs to Developers, Platform Engineers, and Product Managers. 
A solid understanding of SQL can assist in numerous scenarios in your role.

Please work on your own, and if you get completely stuck, feel free to reach out to the facilitator (in #bls-skills-builder on slack) rather than other team members.

Manage your time wisely, you have up to 90 minutes for 11 questions. What you do not complete at the end of the 90 mins you can mark as not done and submit the form. You must submit the form within 90 minutes, but you can continue with the parts you do not get to in your own time. 

All answers can be submitted here, a copy of your submission will be sent to you via email on completion: 
https://forms.gle/HvwyWfwMpK17L4A19

If you finish early, we have a few bonus questions at the end of this doc to push you a little further with some more advanced challenges. These bonus questions can optionally be done in your own time, and you can submit answers to a different form linked below if you would like to. 


**Instructions for Query Tool**:  
You may use any SQL client of your choice to connect to the database. 
If you don’t already have one installed, you can use options like **MySQL Workbench** on Windows, or **Sequel Ace** on Mac. Others suggestions include **DBeaver** or  **Tableplus** or some IDE's have this functionality built in directly.  

If you don't even know what MySQL is, or what a MySql client is, I suggest you start with a ChatGPT conversation to coach you through a basic explanation and some guidelines on getting the client set up.  Use this time as a self study session for your own level with a goal of understanding and solving as many questions as possible. 

Where neccesary, test your queries in the tool before submitting answers. You'll have read only access so no writes, updates or changes to table structure will be possible.  If you encounter issues, troubleshooting and resolving them is part of the challenge!

At the end of this session, you can keep the convo alive in #bls-skills-builder

---

**Let’s dive in and see how far your skills—and the tools at your disposal—can take you.**

### Marketplace Database Schema

Below is a database schema for a simple online marketplace. The marketplace allows users to create listings for items they want to sell, and other users can view these listings and express interest by leaving a message (lead) for the seller. The schema also includes user activity logging for auditing purposes.

We've got a bunch of questions for you to answer based on this database schema. Assume its all MySql 8.0.
 
Below are the credentials you can use to connect and run queries on the database 
- **Host:** `sandbox.cwvzrdsmrbyu.eu-west-1.rds.amazonaws.com`
- **Username:** `bls`
- **Password:** `q3G42vcoUSqnfAbTsfIXbsOmcnLHYjXemBcyH9NW`
- **Database:** `marketplace-skill-builder-challenge`
- **Port:** `3306`

This data is all faked, generated, made up data, and only serves the purpose of giving you a challenge to work on.
Don't look too closely at the data, but rather understand the schema and relationships between the tables. 

The challenge includes questions related to:

1. Connecting and retrieving basic data from the database.
2. Writing SQL queries to answer "real world" questions about the database marketplace data.
3. Debugging and Fixing database queries.

---
### **Database Schema**

#### 1\. **`users`** Table

Stores all user information.

| Column          | Type | Description                                              |
|-----------------| --- |----------------------------------------------------------|
| `user_id`       | `BIGINT UNSIGNED` | Primary Key. Unique identifier for each user.            |
| `username`      | `VARCHAR(50)` | The username of the user.                                |
| `email`         | `VARCHAR(100)` | Unique email address of the user.                        |
| `password_hash` | `VARCHAR(255)` | Hashed password for security.                            |
| `year_of_birth` | `VARCHAR(255)` | Birth Year to calculate their age. |
| `created_at`    | `TIMESTAMP` | Timestamp when the user was created.                     |
| `updated_at`    | `TIMESTAMP` | Timestamp when the user was last updated.                |

-   **Indexes**:
   -   `PRIMARY KEY (user_id)`
   -   `UNIQUE (email)` to ensure email uniqueness.
   -  `INDEX (year_of_birth)`

* * *

#### 2\. **`listings`** Table

Contains all marketplace listings.

| Column | Type | Description |
| --- | --- | --- |
| `listing_id` | `BIGINT UNSIGNED` | Primary Key. Unique identifier for each listing. |
| `user_id` | `BIGINT UNSIGNED` | Foreign Key to `users`. The user who created the listing. |
| `category_id` | `BIGINT UNSIGNED` | Foreign Key to `categories`. The category for the listing. |
| `title` | `VARCHAR(255)` | Title of the listing. |
| `description` | `TEXT` | Detailed description of the listing. |
| `price` | `DECIMAL(10, 2)` | Price of the listing. |
| `status` | `ENUM('active', 'sold', 'expired')` | Status of the listing (active, sold, expired). |
| `created_at` | `TIMESTAMP` | Timestamp when the listing was created. |
| `updated_at` | `TIMESTAMP` | Timestamp when the listing was last updated. |

-   **Indexes**:
   -   `PRIMARY KEY (listing_id)`
   -   `INDEX (user_id)` for efficient user-listing joins.
   -   `INDEX (category_id)` for categorization queries.
* * *

#### 3\. **`leads`** Table

Tracks all leads generated for listings.

| Column | Type | Description |
| --- | --- | --- |
| `lead_id` | `BIGINT UNSIGNED` | Primary Key. Unique identifier for each lead. |
| `listing_id` | `BIGINT UNSIGNED` | Foreign Key to `listings`. The listing the lead is associated with. |
| `user_id` | `BIGINT UNSIGNED` | Foreign Key to `users`. The user who created the lead. Nullable for guest users. |
| `guest_name` | `VARCHAR(100)` | Name of the guest user (nullable if a registered user creates the lead). |
| `guest_email` | `VARCHAR(100)` | Email of the guest user (nullable if a registered user creates the lead). |
| `message` | `TEXT` | Message left by the lead. |
| `created_at` | `TIMESTAMP` | Timestamp when the lead was created. |

-   **Indexes**:
   -   `PRIMARY KEY (lead_id)`
   -   `INDEX (listing_id)` for finding leads by listing.
   -   `INDEX (user_id)` for querying leads by user.
   -   `INDEX (listing_id, lead_id)` to optimize performance when querying for leads.
* * *

#### 4\. **`categories`** Table

Manages categories and subcategories for listings.

| Column | Type | Description |
| --- | --- | --- |
| `category_id` | `BIGINT UNSIGNED` | Primary Key. Unique identifier for each category. |
| `name` | `VARCHAR(255)` | Name of the category. |
| `parent_category_id` | `BIGINT UNSIGNED` | Foreign Key to `categories`. Allows subcategories. Nullable for root categories. |
| `created_at` | `TIMESTAMP` | Timestamp when the category was created. |
| `updated_at` | `TIMESTAMP` | Timestamp when the category was last updated. |

-   **Indexes**:
   -   `PRIMARY KEY (category_id)`
   -   `INDEX (parent_category_id)` for hierarchical queries.
* * *

#### 5\. **`user_activity_log`** Table

Logs user activity for auditing purposes.

| Column | Type | Description |
| --- | --- | --- |
| `log_id` | `BIGINT UNSIGNED` | Primary Key. Unique identifier for each log entry. |
| `user_id` | `BIGINT UNSIGNED` | Foreign Key to `users`. The user performing the action. |
| `activity_type` | `VARCHAR(255)` | Describes the type of activity (e.g., 'login', 'listing\_view'). |
| `activity_details` | `JSON` | JSON field containing additional details about the activity (e.g., listing viewed). |
| `timestamp` | `TIMESTAMP` | Timestamp when the activity was logged. |

-   **Indexes**:
   -   `PRIMARY KEY (log_id)`
   -   `INDEX (user_id)` for querying activities by user.
   -   `INDEX (user_id, timestamp)` for performance optimisation in activity queries.



## **Questions**

---

### **Question 1**: Retrieve the Latest Listing ID and Title

**Question**: Connect to the database and write a query to retrieve the most recent `listing_id` and `title` from the `listings` table. Order the result by `listing_id` in descending order and return the top result.

---

### **Question 2**: Retrieve All Active Listings with Their User Information

**Question**: Write a query to retrieve all listings (ID and Title) that are currently active (`status = 'active'`) and include the username and email of the user who created the listing.

---

### **Question 3**: Retrieve the Number of Leads Per Listing

**Question**: Write a query to return the `listing_id`, `title`, and the total number of leads for each listing. The result should include all listings, even those that have no leads.

---
### **Question 4**: Retrieve Recent User Activity for a Specific User 

**Question**: Write a query to retrieve the most recent 10 activities of a user, including the `activity_type`, `timestamp` and `listing_id` from the `activity_details` column. Assume the `user_id` is 100

---


### **Question 5**: Find Listings in a Specific Category and Subcategories

**Question**: Write a query to return all listings in the category "3" (including any subcategories under 3). Assume an unknown depth category tree and include all category children in your query.  



---

### **Question 6**: Analyze and Improve Query Performance

**Question**: The below query misses the indexes defined and needs to be optimised:
```mysql
SELECT user_id, username FROM users WHERE year_of_birth = 1956
```
Describe how you would investigate and prove that this misses the index. 
Describe the change your would make to ensure that it hits the index. 

---


### **Question 7**: Debug and Fix a Query for Active Listings with Lead Count

**Question**: The following query is supposed to return all active listings (`status = 'active'`) along with the total number of leads for each listing. However, the query is not counting leads correctly for listings that have no leads. Identify and fix the issue, and ensure the results are ordered by the number of leads in descending order.

```mysql
SELECT listings.listing_id, listings.title, COUNT(leads.lead_id) AS total_leads
FROM listings
JOIN leads ON listings.listing_id = leads.listing_id
WHERE listings.status = 'active'
GROUP BY listings.listing_id, listings.title
ORDER BY total_leads DESC;
```
---

### **Question 8**: Debug and Fix a Query for Leads Submitted by Guest and Registered Users

**Question**: The following query is supposed to return the total number of leads submitted by **guest users** (those with `NULL` in `user_id`) and **registered users** for each listing. Additionally, the query should return the name of the user who submitted each lead, and for guest users, it should display `"Guest"` instead of `NULL`. However, the query is not working as expected. Identify and fix the issue.

```mysql
SELECT listings.listing_id, listings.title, COUNT(leads.lead_id) AS total_leads
FROM listings
LEFT JOIN leads ON listings.listing_id = leads.listing_id
GROUP BY listings.listing_id, listings.title;
```

---

### **Question 9**: Get the Average Time Between Listing Creation and First Lead

**Question**: Write a query to calculate the **average time (in hours)** between the creation of a listing and the submission of its first lead. The result should be grouped by listing.


---
### **Question 10**: Define All Required Indexes to Optimize the Query

The following query retrieves detailed information for users who have at least one active listing, including the total number of leads for those listings, the average price of their active listings, and the most recent activity from the users. 
Lets assume that no indexes exist on this table yet. The query will be slow and inefficient because no indexes are defined.

Analyze the query, identify all missing indexes, and write the SQL commands to create the necessary indexes that will significantly improve performance.

---

#### **Query**:
```mysql
SELECT 
    users.user_id, 
    users.username, 
    COUNT(DISTINCT leads.lead_id) AS total_leads, 
    AVG(listings.price) AS avg_price,
    MAX(user_activity_log.timestamp) AS last_user_activity
FROM users
JOIN listings ON users.user_id = listings.user_id
JOIN leads ON listings.listing_id = leads.listing_id
LEFT JOIN user_activity_log ON users.user_id = user_activity_log.user_id
WHERE listings.status = 'active'
GROUP BY users.user_id, users.username;
```

### **Question 11**: Locate and Terminate a Long-Running Query

You’ve been alerted that there is a long-running query that is impacting the performance of the database. Your task is to first locate the query and then terminate it immediately.

Provide the SQL commands that will be used to locate the long-running query and to terminate it.


## Addendum
**For reference, although not required, here is the SQL to create the tables in the database:**

```mysql
-- Create the `users` table 
CREATE TABLE users (
    user_id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
	username VARCHAR(50) NOT NULL,
	email VARCHAR(100) NOT NULL UNIQUE,
	year_of_birth VARCHAR(255) NOT NULL,
	password_hash VARCHAR(255) NOT NULL,
	created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
	updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
                                                          );

-- Create the `categories` table
CREATE TABLE categories (
                            category_id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
                            name VARCHAR(255) NOT NULL,
                            parent_category_id BIGINT UNSIGNED NULL,
                            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                            updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
                            CONSTRAINT fk_parent_category FOREIGN KEY (parent_category_id) REFERENCES categories (category_id) ON DELETE SET NULL
);

-- Create the `listings` table
CREATE TABLE listings (
                          listing_id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
                          user_id BIGINT UNSIGNED NOT NULL,
                          category_id BIGINT UNSIGNED NULL,
                          title VARCHAR(255) NOT NULL,
                          description TEXT,
                          price DECIMAL (10,
                                         2) NOT NULL,
                          status ENUM ('active',
                              'sold',
                              'expired') DEFAULT 'active',
                          created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                          updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
                          CONSTRAINT fk_user_listing FOREIGN KEY (user_id) REFERENCES users (user_id) ON DELETE CASCADE,
                          CONSTRAINT fk_category_listing FOREIGN KEY (category_id) REFERENCES categories (category_id) ON DELETE SET NULL
);

-- Create the `leads` table
CREATE TABLE leads (
                       lead_id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
                       listing_id BIGINT UNSIGNED NOT NULL,
                       user_id BIGINT UNSIGNED NULL,
                       guest_name VARCHAR(100) NULL,
                       guest_email VARCHAR(100) NULL,
                       message TEXT NOT NULL,
                       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                       CONSTRAINT fk_listing_lead FOREIGN KEY (listing_id) REFERENCES listings (listing_id) ON DELETE CASCADE,
                       CONSTRAINT fk_user_lead FOREIGN KEY (user_id) REFERENCES users (user_id) ON DELETE SET NULL
);

-- Create the `user_activity_log` table
CREATE TABLE user_activity_log (
                                   log_id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
                                   user_id BIGINT UNSIGNED NOT NULL,
                                   activity_type VARCHAR(255) NOT NULL,
                                   activity_details JSON,
                                   timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                                   CONSTRAINT fk_user_activity FOREIGN KEY (user_id) REFERENCES users (user_id) ON DELETE CASCADE
);

-- Indexes for performance optimisation
CREATE INDEX idx_user_email ON users (email);

CREATE INDEX idx_user_year_of_birth ON users (year_of_birth);

CREATE INDEX idx_listing_user ON listings (user_id);

CREATE INDEX idx_listing_category ON listings (category_id);

CREATE INDEX idx_lead_listing ON leads (listing_id);

CREATE INDEX idx_lead_user ON leads (user_id);

CREATE INDEX idx_category_parent ON categories (parent_category_id);

CREATE INDEX idx_activity_user ON user_activity_log (user_id);

CREATE INDEX idx_listing_lead ON leads (listing_id, lead_id);

CREATE INDEX idx_user_activity_timestamp ON user_activity_log (user_id, timestamp);



```



## Bonus Questions: (Answers can be submitted to a new form [here](https://forms.gle/j49yecsA4zJCLktNA) anytime during or after the challenge. No time limit)

---

### **Bonus Question 1**: Identify Users with Listings that Have a Lower Conversion Rate Than the 75th Percentile

**Question**: Write a query to identify users whose listings have a **conversion rate** (leads per listing) below the 75th percentile of conversion rates across all listings. The result should include the `user_id`, `username`, total listings, total leads, and the user's conversion rate.
Use the NTILE function to assist with the calculations.
 
---

### **Bonus Question 2**:

**Question**: Write a query that returns all listings (title, and listing ID) which do not have any leads. 
 
---

### **Bonus Question 3**: 

**Question**: Find Users Who Logged in on Two Consecutive Days. Return their name and ID. 
 
---

### **Bonus Question 4**: 

**Question**: Find the Time Difference Between User Registration and Their First Lead. Return the User ID, Name and the time difference for each user. 
 
---

### **Bonus Question 5**: 

**Question**: More recent MySQL version have features that could improve querying. Look at the below queries and rewrite each one to be more modern and efficient. 


A)  Find Categories with More Listings Than the Average
```
SELECT c.*
FROM categories c
WHERE (SELECT COUNT(*) FROM listings l WHERE l.category_id = c.id) > (
    SELECT AVG(listing_count)
    FROM (SELECT COUNT(*) AS listing_count FROM listings GROUP BY category_id) AS avg_table
);
```

B)  Find Listings with the Most Leads in Each Category
```
SELECT l.id, l.title, l.category_id
FROM listings l
WHERE l.id IN (
    SELECT lead.listing_id
    FROM leads lead
    GROUP BY lead.listing_id
    HAVING COUNT(lead.id) = (
        SELECT MAX(lead_count)
        FROM (SELECT listing_id, COUNT(*) AS lead_count FROM leads GROUP BY listing_id) AS lead_counts
        WHERE lead_counts.listing_id = l.id
    )
);

```

C) Find Users Who Logged in on Two Consecutive Days

```
SELECT u1.user_id, u1.activity_time
FROM user_activities_log u1
JOIN user_activities_log u2 ON u1.user_id = u2.user_id
WHERE DATE(u2.activity_time) = DATE(u1.activity_time) + INTERVAL 1 DAY
```
