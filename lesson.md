# **Lesson Plan: Advanced SQL for Business Insights**

## **Part 1: The Map and the Bridge (Joins & Unions) 

### **Learning Objectives**

* Navigate database schemas using Meta Queries.  
* Combine data using 4 Join types and Unions.

### **Theory Recap**

**The Analogy:** Imagine you are at a party. You have a list of names (client) and a list of who brought which gift (claim).

* **Inner Join:** Only people at the party who brought a gift.  
* **Left Join:** Everyone at the party; if they didn't bring a gift, the "gift" column is just an empty box (NULL).  
* **Union:** Putting two lists of names (e.g., Employees and Contractors) into one long "Staff" list.

### **Workshop**

**Instructor Script:** "Before we build the bridge, we need to know the terrain. Let's look at our 'Card Catalog'."

```sql
-- Meta Queries: Knowing your environment  
SHOW TABLES;   
DESCRIBE client;  
SUMMARIZE claim;
```

**Socratic Prompt:** "If I SUMMARIZE the claim table and see a max(claim_amt) that is 10x higher than the average, what does that tell you about our insurance risk?"

Hands-on Exercise 1:  
Create a master report of every claim. Include the client's name, their car type, and the city they live in.  
Hint: You will need to join 4 tables.  

```sql
-- Solution for Master Report  
SELECT   
    cl.id, cl.claim_date, cl.claim_amt,  
    c.car_type,  
    cli.first_name, cli.last_name,  
    a.city, a.state  
FROM claim cl  
INNER JOIN car c ON cl.car_id = c.id  
INNER JOIN client cli ON cl.client_id = cli.id  
INNER JOIN address a ON cli.address_id = a.id;  
```

### **Q\&A / Reflection**

Anticipated Hurdle: Learners often struggle with "Which table is Left?".  
Solution: Always think of the "Primary Subject" as the Left table.  
Business Case: How would a "Full Outer Join" help us find cars that have never been claimed AND claims that (erroneously) don't have a car attached?

## **Part 2: The Moving Window (Window Functions)**

### **Learning Objectives**

* Calculate running totals and rankings without losing row detail.

### **Theory Recap**

The Analogy:  
Standard GROUP BY is like taking a whole class and saying "The average height is 5'8"." You lose the individuals.  
A Window Function is like walking down the line of students and saying "You are the 1st tallest, you are the 2nd tallest..." while they all remain standing in line.

### **Workshop**

**Instructor Script:** "Let's find our 'Heavy Hitters'. Who has the highest claims per car category?"

```sql

-- The Ranking Window  
SELECT   
    id, car_id, claim_amt,  
    RANK() OVER (PARTITION BY car_id ORDER BY claim_amt DESC) AS rank  
FROM claim  
QUALIFY rank <= 3; -- DuckDB specific 'QUALIFY' to filter windows  
```

Hands-on Exercise 2:  
Calculate a running total of insurance payouts over time (ordered by claim_date).  

```sql
-- Solution for Running Total  
SELECT   
    claim_date, claim_amt,  
    SUM(claim_amt) OVER (ORDER BY claim_date) AS running_total  
FROM claim;  
```

## **Part 3: Nested Logic (Subqueries & CTEs)**

### **Learning Objectives**

* Simplify complex logic using Common Table Expressions (CTEs).

### **Theory Recap**

The Analogy:  
A Subquery is like a "thought within a thought."  
A CTE (Common Table Expression) is like writing down a recipe step before you start cooking. It makes the code readable for humans, not just machines.

### **Workshop**

**Instructor Script:** "Let's find cars whose resale value is below the average for their specific type. It sounds complex, but we'll build it layer by layer."

```sql
-- The CTE Approach (Clean and Readable)  
WITH AvgValues AS (  
    SELECT car_type, AVG(resale_value) as avg_resale  
    FROM car  
    GROUP BY car_type  
)  
SELECT c.id, c.car_type, c.resale_value, a.avg_resale  
FROM car c  
JOIN AvgValues a ON c.car_type = a.car_type  
WHERE c.resale_value < a.avg_resale;
```

Hands-on Exercise 3:  
Find clients who have made claims that are more than 50% of their annual income. Use a CTE to calculate the total claims per client first.

### **Q\&A / Reflection**

**Reflection:** Why do developers prefer CTEs over Subqueries? (Answer: Readability and Debugging).
