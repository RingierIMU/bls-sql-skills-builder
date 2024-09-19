# MySQL Skill Builder Challenge. 


### **Question 1**: Retrieve the Latest Listing ID and Title

**Question**: Connect to the database and write a query to retrieve the most recent `listing_id` and `title` from the `listings` table. Order the result by `listing_id` in descending order and return the top result.

--- 
**Answer**:

**Expected SQL**:
```mysql
SELECT listing_id, title
FROM listings
ORDER BY listing_id DESC
LIMIT 1;
```


### **Question 2**: Retrieve All Active Listings with Their User Information

**Question**: Write a query to retrieve all listings (ID and Title) that are currently active (`status = 'active'`) and include the username and email of the user who created the listing.

--- 
**Answer**:

**Expected SQL**:


```mysql
SELECT
    listings.listing_id,
    listings.title,
    users.username,
    users.email
FROM
    listings
        JOIN users ON listings.user_id = users.user_id
WHERE
    listings.status = 'active';

```

* * *

### **Question 3**: Retrieve the Number of Leads Per Listing

**Question**: Write a query to return the `listing_id`, `title`, and the total number of leads for each listing. The result should include all listings, even those that have no leads.

--- 
**Answer**:

**Expected SQL**:
```mysql
SELECT
	listings.listing_id,
	listings.title,
	COUNT(leads.lead_id) AS total_leads
FROM
	listings
	LEFT JOIN leads ON listings.listing_id = leads.listing_id
GROUP BY
	listings.listing_id,
	listings.title;
```


---
### **Question 4**: Retrieve Recent User Activity for a Specific User 

**Question**: Write a query to retrieve the most recent 10 activities of a user, including the `activity_type`, `timestamp` and `listing_id` from the `activity_details` column. Assume the `user_id` is 100

---

**Answer**:

**Expected SQL**:

```mysql
SELECT
    user_activity_log.activity_type,
    user_activity_log.timestamp,
    JSON_EXTRACT(user_activity_log.activity_details, '$.listing_id') AS listing_id
FROM
    user_activity_log
WHERE
    user_activity_log.user_id = 100
ORDER BY
    user_activity_log.timestamp DESC
LIMIT 10;
```

-   **JSON\_EXTRACT** is used to extract the `listing_id` from the `activity_details` JSON column.
-   The JSON extraction will work for activities that have a `listing_id` key in the JSON data (such as `listing_view` activities). If the key doesn’t exist, the result will return `NULL`.


* * *

### **Question 5**: Find Listings in a Specific Category and Subcategories

**Question**: Write a query to return all listings in the category "3" and any subcategories under 3. Assume an unknown depth category tree and include all category children in your query.  

--- 
**Answer**:

**Expected SQL**:

```mysql
WITH RECURSIVE category_hierarchy AS (
    SELECT
        category_id
    FROM
        categories
    WHERE
        category_id = 3 
    UNION ALL
    SELECT
        categories.category_id
    FROM
        categories
            JOIN category_hierarchy ON categories.parent_category_id = category_hierarchy.category_id
)
SELECT
    listings.listing_id,
    listings.title,
    listings.price
FROM
    listings
        JOIN category_hierarchy ON listings.category_id = category_hierarchy.category_id;

```

---

### **Question 6**: Analyze and Improve Query Performance

**Question**: The below query misses the indexes defined and needs to be optimised:
```mysql
SELECT user_id, username FROM users WHERE year_of_birth = 1956
```
Describe how you would investigate and prove that this misses the index. 
Describe how to change thid query so that it hits the index

---
**Answer:**

To investigate and prove that the query misses the index, and to modify it so that it utilizes the index, follow these steps:

---

#### Part 1. Investigate the Query Execution Plan

**Use the `EXPLAIN` Statement:**

Run the following command to see how MySQL executes the query:

```sql
EXPLAIN SELECT user_id, username FROM users WHERE year_of_birth = 1956;
```

**Analyze the Output:**

The `EXPLAIN` statement provides details about how MySQL executes the query, including whether it uses indexes. Key columns to focus on are:

- **`type`**: Indicates the type of access method used. `ALL` means a full table scan.
- **`possible_keys`**: Shows which indexes MySQL could potentially use.
- **`key`**: The actual index MySQL decides to use.

**Possible Output Indicating Index Not Used:**

| id | select_type | table | type | possible_keys     | key  | key_len | ref  | rows  | Extra       |
|----|-------------|-------|------|-------------------|------|---------|------|-------|-------------|
| 1  | SIMPLE      | users | ALL  | idx_year_of_birth | NULL | NULL    | NULL | 10000 | Using where |

- **Interpretation:**
   - **`type: ALL`**: Indicates a full table scan.
   - **`possible_keys: idx_year_of_birth`**: Shows that an index on `year_of_birth` exists.
   - **`key: NULL`**: Indicates that the index is not being used.

---

#### Part 2. Understand Why the Index Is Not Used

**Data Type Mismatch:**

- The `year_of_birth` column is defined as `VARCHAR(255)`.
- The query uses an unquoted numeric value `1956`.
- MySQL may perform implicit type conversion, which can prevent the use of the index.

**Implicit Conversion:**

- Comparing a string column to a numeric value causes MySQL to convert the column values to numbers during the comparison.
- This conversion can disable the use of the index because it must process each row to perform the conversion.

---

#### Part 3. Modify the Query to Use the Index

**Match Data Types by Quoting the Value:**

Since `year_of_birth` is a string (`VARCHAR`), the value in the `WHERE` clause should be a string literal.

**Rewritten Query:**

```sql
SELECT user_id, username FROM users WHERE year_of_birth = '1956';
```

**Verify Index Usage with `EXPLAIN`:**

Run `EXPLAIN` on the modified query:

```sql
EXPLAIN SELECT user_id, username FROM users WHERE year_of_birth = '1956';
```

**Expected Output Indicating Index Used:**

| id | select_type | table | type | possible_keys     | key               | key_len | ref   | rows | Extra       |
|----|-------------|-------|------|-------------------|-------------------|---------|-------|------|-------------|
| 1  | SIMPLE      | users | ref  | idx_year_of_birth | idx_year_of_birth | 767     | const | 50   | Using where |

- **Interpretation:**
   - **`type: ref`**: Indicates that the index is being used for a non-unique scan.
   - **`key: idx_year_of_birth`**: Confirms that the index on `year_of_birth` is utilized.
   - **`rows`**: Shows a reduced number of rows being examined, improving performance.

---

#### Part 4. Consider Changing the Data Type (Optional)

For better performance and data integrity, consider changing the `year_of_birth` column to a numeric data type.

**Change Data Type to `YEAR` or `SMALLINT`:**

- **Option 1: Use `YEAR` Data Type**

  ```sql
  ALTER TABLE users MODIFY year_of_birth YEAR;
  ```

- **Option 2: Use `SMALLINT`**

  ```sql
  ALTER TABLE users MODIFY year_of_birth SMALLINT UNSIGNED;
  ```

**Update the Index (If Necessary):**

If you change the data type, ensure the index is still appropriate:

```sql
ALTER TABLE users DROP INDEX idx_year_of_birth;
CREATE INDEX idx_year_of_birth ON users(year_of_birth);
```

**Modify the Query Accordingly:**

If `year_of_birth` is now numeric, you can use the value without quotes:

```sql
SELECT user_id, username FROM users WHERE year_of_birth = 1956;
```

**Verify with `EXPLAIN`:**

```sql
EXPLAIN SELECT user_id, username FROM users WHERE year_of_birth = 1956;
```

**Expected Output:**

| id | select_type | table | type | possible_keys     | key               | key_len | ref   | rows | Extra       |
|----|-------------|-------|------|-------------------|-------------------|---------|-------|------|-------------|
| 1  | SIMPLE      | users | ref  | idx_year_of_birth | idx_year_of_birth | 2       | const | 50   | Using where |

- **Interpretation:**
   - The index is used with a smaller `key_len`, indicating more efficient storage and comparison.



----


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
**Answer**:

**Corrected Query**:
```mysql
SELECT listings.listing_id, listings.title, COUNT(leads.lead_id) AS total_leads
FROM listings
LEFT JOIN leads ON listings.listing_id = leads.listing_id
WHERE listings.status = 'active'
GROUP BY listings.listing_id, listings.title
ORDER BY total_leads DESC;
```

**Explanation**:
- **LEFT JOIN**: Using a `LEFT JOIN` ensures that all active listings are included, even if they have no leads.
- **COUNT(leads.lead_id)**: The `COUNT` function returns `0` for listings with no leads.
- **ORDER BY total_leads DESC**: The query now orders the results by `total_leads` in descending order, showing listings with the most leads first.


### **Question 9**: Get the Average Time Between Listing Creation and First Lead

**Question**: Write a query to calculate the **average time (in hours)** between the creation of a listing and the submission of its first lead. The result should be grouped by listing.

--- 
**Answer**:

**Expected SQL**:
```mysql
SELECT 
    listings.listing_id, 
    listings.title, 
    TIMESTAMPDIFF(HOUR, listings.created_at, MIN(leads.created_at)) AS hours_to_first_lead
FROM listings
JOIN leads ON listings.listing_id = leads.listing_id
GROUP BY listings.listing_id, listings.title;
```

**Explanation**:
- **MIN(leads.created_at)**: Retrieves the timestamp of the first lead for each listing.
- **TIMESTAMPDIFF(HOUR, ...)**: Calculates the difference in hours between the listing creation time (`listings.created_at`) and the first lead time (`MIN(leads.created_at)`).
- **GROUP BY**: Groups the result by each listing to calculate the time difference per listing.


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

--- 
**Answer**:


The performance issues in this query stem from several **JOINs** and filtering conditions, all of which require proper indexing to ensure optimal performance. Specifically:
1. **JOINs** between `users`, `listings`, `leads`, and `user_activity_log` require indexes on the foreign key columns.
2. Filtering on `listings.status = 'active'` needs to be optimized with an index on the `status` column.
3. Efficient retrieval of recent activity from `user_activity_log` requires an index on `timestamp`.

---

### **Required Indexes**:

1. **Index on `listings.status`**:
    - This index optimizes the filtering of listings by their `status` (`'active'`).
   ```mysql
   CREATE INDEX idx_listing_status ON listings(status);
   ```

2. **Index on `listings.user_id`**:
    - This index optimizes the **JOIN** between `users` and `listings`, ensuring fast retrieval of listings for each user.
   ```mysql
   CREATE INDEX idx_listing_user ON listings(user_id);
   ```

3. **Index on `leads.listing_id`**:
    - This index optimizes the **JOIN** between `listings` and `leads`, speeding up the counting of leads for each listing.
   ```mysql
   CREATE INDEX idx_leads_listing ON leads(listing_id);
   ```

4. **Index on `user_activity_log.user_id, user_activity_log.timestamp`**:
    - This **compound index** allows MySQL to quickly find the most recent activity for each user. Without this index, MySQL will scan the entire `user_activity_log` table for each user, which could be very inefficient.
   ```mysql
   CREATE INDEX idx_user_activity ON user_activity_log(user_id, timestamp);
   ```

5. **Index on `users.user_id`**:
    - Although `user_id` is a primary key, confirming the existence of an index on this column will help optimize the **JOIN** and **GROUP BY** operations.
   ```mysql
   CREATE INDEX idx_user_id ON users(user_id);
   ```

---

### **Explanation**:
- **Index on `listings(status)`**: This index allows MySQL to filter out inactive listings efficiently.
- **Index on `listings(user_id)`**: Optimizes the **JOIN** between `listings` and `users` to ensure fast access to a user's listings.
- **Index on `leads(listing_id)`**: Improves performance when counting leads for each listing.
- **Compound Index on `user_activity_log(user_id, timestamp)`**: This index allows MySQL to quickly locate the most recent activity for each user by first filtering by `user_id` and then sorting by `timestamp`.
- **Index on `users(user_id)`**: Ensures efficient grouping and joining when retrieving user data.

---

### **Question 11**: Locate and Terminate a Long-Running Query

You’ve been alerted that there is a long-running query that is impacting the performance of the database. Your task is to first locate the query and then terminate it immediately.

Provide the SQL commands that will be used to locate the long-running query and to terminate it.
 
--- 
**Answer**:

1. **Locate the Long-Running Query**:
   ```mysql
   SHOW PROCESSLIST;
   ```

2. Once you have identified the query with its **`ID`**, execute:
   ```mysql
   KILL QUERY <query_id>;
   ```


