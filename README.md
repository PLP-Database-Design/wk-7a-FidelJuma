# 📝 Assignment: Database Design and Normalization

## 🎯 **Learning Objectives**
* 🛠️ **Understand the principles of good database design** and **normalization**.
* 💡 **Apply normalization techniques** to improve database structure and efficiency.
* 🔍 **Learn First, Second, and Third Normal Forms** (1NF, 2NF, 3NF) to eliminate redundancy and optimize data storage.

---

## 📋 **What You'll Need**
* 💻 A computer with internet access.
* ✍️ A code editor (e.g., Visual Studio Code).
* 🖥️ MySQL Workbench or another SQL database environment.

---


## 📝 Submission Instructions  
📂 Write all your SQL queries in the **answers.sql** file.  
✍️ Answer each question concisely and make sure your queries are clear and correct.  
🗣️ Structure your responses clearly, and use comments if necessary to explain your approach.

--- 

## 📚 Assignment Questions

---

### Question 1 Achieving 1NF (First Normal Form) 🛠️
Task:
- You are given the following table **ProductDetail**:

| OrderID | CustomerName  | Products                        |
|---------|---------------|---------------------------------|
| 101     | John Doe      | Laptop, Mouse                   |
| 102     | Jane Smith    | Tablet, Keyboard, Mouse         |
| 103     | Emily Clark   | Phone                           |


- In the table above, the **Products column** contains multiple values, which violates **1NF**.
- **Write an SQL query** to transform this table into **1NF**, ensuring that each row represents a single product for an order
To transform the given table into **1NF** (First Normal Form), you need to ensure that each row contains only a single value in the **Products** column, meaning there should be one row for each product in an order.

You can achieve this by using a method to "explode" or "unpivot" the **Products** column into individual rows.

Here’s an SQL query to achieve this transformation:

```sql
SELECT OrderID, CustomerName, TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(Products, ',', n.n), ',', -1)) AS Product
FROM ProductDetail
JOIN (SELECT 1 AS n UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5) n
  ON CHAR_LENGTH(Products) - CHAR_LENGTH(REPLACE(Products, ',', '')) >= n.n - 1
ORDER BY OrderID, n.n;
```

### Explanation:
- We join the **ProductDetail** table with a derived table (a list of numbers) that allows us to break up the comma-separated **Products** column.
- The `SUBSTRING_INDEX` function is used to extract individual products.
- `TRIM()` is used to clean up any extra spaces.
- The `JOIN` ensures that each product from the comma-separated list is split into its own row.
- The `ORDER BY` ensures the rows are ordered by **OrderID** and the position of the product in the list.

This query transforms the table so that each product appears in its own row, complying with **1NF**. 

The resulting table will look like this:

| OrderID | CustomerName | Product  |
|---------|--------------|----------|
| 101     | John Doe     | Laptop   |
| 101     | John Doe     | Mouse    |
| 102     | Jane Smith   | Tablet   |
| 102     | Jane Smith   | Keyboard |
| 102     | Jane Smith   | Mouse    |
| 103     | Emily Clark  | Phone    |

Each row now represents a single product per order, satisfying the **1NF** requirement.
--- 

### Question 2 Achieving 2NF (Second Normal Form) 🧩

- You are given the following table **OrderDetails**, which is already in **1NF** but still contains partial dependencies:

| OrderID | CustomerName  | Product      | Quantity |
|---------|---------------|--------------|----------|
| 101     | John Doe      | Laptop       | 2        |
| 101     | John Doe      | Mouse        | 1        |
| 102     | Jane Smith    | Tablet       | 3        |
| 102     | Jane Smith    | Keyboard     | 1        |
| 102     | Jane Smith    | Mouse        | 2        |
| 103     | Emily Clark   | Phone        | 1        |

- In the table above, the **CustomerName** column depends on **OrderID** (a partial dependency), which violates **2NF**. 

- Write an SQL query to transform this table into **2NF** by removing partial dependencies. Ensure that each non-key column fully depends on the entire primary key.
To transform the **OrderDetails** table into **2NF** (Second Normal Form), we need to remove partial dependencies. In **2NF**, all non-key columns must depend on the whole primary key, not just a part of it. In this case, **OrderID** and **Product** together form the primary key (since each order can contain multiple products), and **CustomerName** is partially dependent on **OrderID**.

### Steps to normalize the table to **2NF**:

1. **Identify Partial Dependencies**:
   - **CustomerName** depends only on **OrderID**, but not on the **Product**.
   - This creates a partial dependency, violating **2NF**.

2. **Decompose the table into two**:
   - Create a new table to store information about the **OrderID** and **CustomerName**, so that **CustomerName** is no longer dependent on the product.
   - The original table should only store data related to the specific product and quantity, removing the partial dependency.

### The new tables in **2NF**:

1. **Orders** table (to store OrderID and CustomerName):

| OrderID | CustomerName  |
|---------|---------------|
| 101     | John Doe      |
| 102     | Jane Smith    |
| 103     | Emily Clark   |

2. **OrderDetails** table (to store Product and Quantity for each OrderID):

| OrderID | Product      | Quantity |
|---------|--------------|----------|
| 101     | Laptop       | 2        |
| 101     | Mouse        | 1        |
| 102     | Tablet       | 3        |
| 102     | Keyboard     | 1        |
| 102     | Mouse        | 2        |
| 103     | Phone        | 1        |

### SQL Queries to Normalize:

#### 1. Create the **Orders** table:
```sql
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerName VARCHAR(100)
);
```

#### 2. Create the **OrderDetails** table:
```sql
CREATE TABLE OrderDetails (
    OrderID INT,
    Product VARCHAR(100),
    Quantity INT,
    PRIMARY KEY (OrderID, Product),
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID)
);
```

#### 3. Insert the data into the **Orders** table:
```sql
INSERT INTO Orders (OrderID, CustomerName)
SELECT DISTINCT OrderID, CustomerName
FROM OrderDetails;
```

#### 4. Insert the data into the **OrderDetails** table:
```sql
INSERT INTO OrderDetails (OrderID, Product, Quantity)
SELECT OrderID, Product, Quantity
FROM OrderDetails;
```

### Now, the data is in **2NF** because:

- The **CustomerName** column is moved to a separate **Orders** table, so it only depends on **OrderID**.
- The **OrderDetails** table now only contains data about the products and quantities, with a primary key of **OrderID** and **Product** combined, ensuring there's no partial dependency.


---
Good luck 🚀
