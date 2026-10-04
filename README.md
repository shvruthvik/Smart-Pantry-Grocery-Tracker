# Project Check-in 1

# SMART PANTRY: GROCERY EXPIRATION & WASTE TRACKER

**Student:** Chiranjeevi Venkata Shiva Ruthvik Savaram\
**Student ID:** Y247K345

------------------------------------------------------------------------

## 1. Problem Definition and Mobile Scope

### 1.1 Problem Definition

People often buy groceries and forget what they already have at home or
when those items will expire. This can result in food being wasted
because items are forgotten until they expire. It can also cause people
to buy groceries that they already have.

For this project, I plan to develop a mobile application called **Smart
Pantry: Grocery Expiration & Waste Tracker**. The application will allow
users to record groceries they have at home and keep track of their
quantity, purchase date, and expiration date.

The main purpose of the application is to help users see what groceries
they have, identify items that have expired, and identify items that
will expire soon. This can help users decide which groceries they should
use first and reduce unnecessary food waste.

### 1.2 Target Users and Platform

The application will be designed for individual users and small
households.

The target platform is **Android**. I will focus on Android instead of
trying to support both Android and iOS so that I can keep the project
manageable during the semester.

### 1.3 Application Scope

The main features I plan to include are:

-   Add a grocery item
-   View grocery items
-   Edit grocery information
-   Delete a grocery item
-   Store quantity, purchase date, and expiration date
-   Assign a category to each grocery
-   Mark an item as consumed or discarded
-   View expired groceries
-   View groceries expiring within the next seven days
-   Filter groceries by category or status

An item is considered expired when its expiration date is before the
current date. Items with an expiration date within the next seven days
are shown as expiring soon. Expired and expiring soon are calculated
from the expiration date instead of being stored in the database, so
they always stay accurate as time passes.

------------------------------------------------------------------------

## 2. Initial Database Design

For Check-in 1, I am starting with two tables: **Users** and
**GroceryItem**. I plan to expand and normalize the database during
Check-in 2 after identifying possible dependencies and problems in this
initial design.

### Users

  Column   Key           Description
  -------- ------------- ------------------------------------------
  UserID   Primary Key   Unique ID for a user
  Name                   User's name
  Email                  User's email (all emails will be unique)

### GroceryItem

  Column           Key           Description
  ---------------- ------------- --------------------------------
  ItemID           Primary Key   Unique ID for a grocery item
  UserID           Foreign Key   User who owns the item
  ItemName                       Name of grocery
  Category                       Grocery category
  Quantity                       Number of items
  PurchaseDate                   Date purchased
  ExpirationDate                 Expiration date
  Status                         Active, Consumed, or Discarded

The relationship is **one-to-many**. One user can have multiple grocery
items, while each grocery item belongs to one user. The `UserID` in the
`GroceryItem` table is a foreign key that references `UserID` in the
`Users` table.

Status only records what the user did with the item (still has it, used
it, or threw it away). It does not store `"Expired"`, because that is
calculated from `ExpirationDate`.

------------------------------------------------------------------------

## 3. SQL and Relational Algebra

The following SQL and relational algebra examples show how the database
will support some of the main features of the application.

For the examples below, I am using **October 4, 2026** as the current
date.

### 3.1 Database Creation

#### SQL Code

``` sql
CREATE TABLE Users (
    UserID INTEGER PRIMARY KEY,
    Name VARCHAR(100) NOT NULL,
    Email VARCHAR(150) NOT NULL UNIQUE
);

CREATE TABLE GroceryItem (
    ItemID INTEGER PRIMARY KEY,
    UserID INTEGER NOT NULL,
    ItemName VARCHAR(100) NOT NULL,
    Category VARCHAR(50) NOT NULL,
    Quantity INTEGER NOT NULL CHECK (Quantity > 0),
    PurchaseDate DATE NOT NULL,
    ExpirationDate DATE NOT NULL,
    Status VARCHAR(20) NOT NULL
        CHECK (Status IN ('Active', 'Consumed', 'Discarded')),
    FOREIGN KEY (UserID) REFERENCES Users(UserID),
    CHECK (ExpirationDate >= PurchaseDate)
);
```

The primary keys uniquely identify records in each table. The foreign
key creates the relationship between the user and the grocery items.

I also added a few simple constraints: emails must be unique, quantity
must be more than zero, status can only be one of three values, and the
expiration date cannot be before the purchase date.

### Sample Data

``` sql
INSERT INTO Users (UserID, Name, Email)
VALUES
(1, 'Ruthvik', 'ruthvik@mail.com'),
(2, 'John', 'john@mail.com');

INSERT INTO GroceryItem
(ItemID, UserID, ItemName, Category, Quantity,
 PurchaseDate, ExpirationDate, Status)
VALUES
(101, 1, 'Spinach', 'Vegetables', 1, '2026-10-01', '2026-10-06', 'Active'),
(102, 1, 'Milk', 'Dairy', 1, '2026-10-02', '2026-10-05', 'Active'),
(103, 1, 'Apples', 'Fruits', 5, '2026-10-01', '2026-10-15', 'Active'),
(104, 2, 'Yogurt', 'Dairy', 2, '2026-10-01', '2026-10-03', 'Active'),
(105, 2, 'Bread', 'Bakery', 1, '2026-10-03', '2026-10-08', 'Active'),
(106, 1, 'Bananas', 'Fruits', 3, '2026-09-28', '2026-10-02', 'Active');
```

------------------------------------------------------------------------

### 3.2 Query 1 --- Display Grocery Information

#### Relational Algebra

$$
\pi_{ItemID,ItemName,Category,Quantity,ExpirationDate}
\left(
\sigma_{UserID=1}(GroceryItem)
\right)
$$

#### SQL

``` sql
SELECT ItemID, ItemName, Category, Quantity, ExpirationDate
FROM GroceryItem
WHERE UserID = 1;
```

**Expected result:** Spinach, Milk, Apples, Bananas

Selection keeps only User 1's items, and projection picks the columns
needed for the grocery list screen.

------------------------------------------------------------------------

### 3.3 Query 2 --- Find Expired Groceries

#### Relational Algebra

$$
\sigma_{
UserID=1 \land
Status='Active' \land
ExpirationDate<'2026-10-04'
}
(GroceryItem)
$$

#### SQL

``` sql
SELECT *
FROM GroceryItem
WHERE UserID = 1
  AND Status = 'Active'
  AND ExpirationDate < '2026-10-04';
```

**Expected result:** Bananas

This finds items the user still has whose expiration date has already
passed. Items that were already consumed or discarded are not shown.

------------------------------------------------------------------------

### 3.4 Query 3 --- Find Groceries Expiring Within Seven Days

#### Relational Algebra

$$
\sigma_{
UserID=1 \land
Status='Active' \land
ExpirationDate\geq'2026-10-04' \land
ExpirationDate\leq'2026-10-11'
}
(GroceryItem)
$$

#### SQL

``` sql
SELECT *
FROM GroceryItem
WHERE UserID = 1
  AND Status = 'Active'
  AND ExpirationDate >= '2026-10-04'
  AND ExpirationDate <= '2026-10-11';
```

**Expected result:** Spinach, Milk

This query supports the **Expiring Soon** feature of the application.

------------------------------------------------------------------------

### 3.5 Query 4 --- Filter Groceries by Category

#### Relational Algebra

$$
\sigma_{
Category='Vegetables' \land
UserID=1
}
(GroceryItem)
$$

#### SQL

``` sql
SELECT *
FROM GroceryItem
WHERE UserID = 1
  AND Category = 'Vegetables';
```

**Expected result:** Spinach

This lets the user view their groceries in one category.

------------------------------------------------------------------------

### 3.6 Query 5 --- Connect Users and Grocery Items

#### Relational Algebra

$$
\pi_{Name,ItemName,Category,ExpirationDate}
\left(
\sigma_{UserID=1}(Users)
\bowtie
GroceryItem
\right)
$$

#### SQL Code

``` sql
SELECT Users.Name,
       GroceryItem.ItemName,
       GroceryItem.Category,
       GroceryItem.ExpirationDate
FROM Users
JOIN GroceryItem
  ON Users.UserID = GroceryItem.UserID
WHERE Users.UserID = 1;
```

**Expected result:** Ruthvik with Spinach, Milk, Apples, and Bananas

This query shows the relationship between the two tables. First,
selection keeps only User 1 from the `Users` table. Then the join
matches that user to their grocery items using the shared `UserID`
column. Finally, projection shows the user's name next to each item.

------------------------------------------------------------------------

## 4. AI Utilization Plan

I plan to use generative AI mainly as a learning and debugging tool
during the project. My main AI tool will be **ChatGPT**. I may also use
Claude or Gemini when I want to compare explanations, verify
documentation, or get another perspective on a problem.

I do not plan to have AI generate the entire project without
understanding it. Instead, I will use it to learn concepts, troubleshoot
errors, review my decisions, and understand code that I am working with.

### Example Uses and Prompts

#### Learning Relational Algebra

> "Explain the difference between selection and projection in relational
> algebra using a small grocery database. Explain how I can decide which
> one to use."

#### SQL Debugging

> "I am getting this SQL error. Explain what the error means and help me
> identify what I should check before giving me the corrected query."

#### Database Design

> "Review my initial grocery database design and point out possible
> duplicates or dependencies that I should investigate. Don't redesign
> it for me yet."

#### Understanding Android/Database Code

> "Explain this Android database code line by line. I want to understand
> how the application is interacting with the database before I make
> changes."

I will verify important AI suggestions by testing the SQL/code myself
and checking documentation when necessary. I will also keep an AI log
throughout the project to record meaningful AI interactions and what I
learned from them.

------------------------------------------------------------------------

## Project Progression

This initial two-table design will serve as the foundation for the next
stages of the project. During **Check-in 2**, I will expand and
normalize the database to 3NF after identifying dependencies and
potential anomalies. During **Check-in 3**, I will integrate the
normalized database with the Android application through the ORM/API
layer and test the complete application.
