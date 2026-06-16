# ⚽ Football Ticket Booking System - Database Schema & Design

A professional-grade PostgreSQL database design and implementation for a Football Ticket Booking System. This project showcases relational database design, schema architecture, entity-relationship cardinality, and rigorous check constraints to maintain referential and data integrity.

---

##  Entity Relationship Diagram (ERD)

The database schema manages three core entities: **Users**, **Matches**, and **Bookings**. Below is the visual representation of their relationships based on.

```mermaid
erDiagram
    Users {
        INT user_id PK
        VARCHAR full_name
        VARCHAR email UNIQUE
        VARCHAR role
        VARCHAR phone_number
    }
    Matches {
        INT match_id PK
        VARCHAR fixture
        VARCHAR tournament_category
        DECIMAL base_ticket_price
        VARCHAR match_status
    }
    Bookings {
        INT booking_id PK
        INT user_id FK
        INT match_id FK
        VARCHAR seat_number
        VARCHAR payment_status
        DECIMAL total_cost
    }
    Users ||--o{ Bookings : "places"
    Matches ||--o{ Bookings : "contains"
```

###  Cardinality & Relationship Constraints
* **One-to-Many (`Users` to `Bookings`):** A registered user can make multiple bookings (`1:N` relationship). Each booking is linked to a single user via the `user_id` Foreign Key.
* **One-to-Many (`Matches` to `Bookings`):** A scheduled match can have many tickets booked under it (`1:N` relationship). Each booking references a specific match via the `match_id` Foreign Key.
* **Many-to-Many Resolution (`Bookings`):** The `Bookings` table acts as a junction entity resolving the relationship between `Users` and `Matches`, storing metadata such as the assigned seat number, payment status, and the total transaction cost.

---

##  Data Dictionary & Schema Constraints

### 1. `Users` Table
Stores contact information and authentication roles for consumers and administrators.

| Column Name | Data Type | Key/Constraint | Description |
| :--- | :--- | :--- | :--- |
| `user_id` | `INT` | `PRIMARY KEY` | Unique identifier for each user. |
| `full_name` | `VARCHAR(100)` | - | First and last name of the user. |
| `email` | `VARCHAR(100)` | `UNIQUE` | Unique, non-duplicable login email. |
| `role` | `VARCHAR(50)` | `CHECK` | Restricted to: `'Ticket Manager'`, `'Football Fan'`. |
| `phone_number` | `VARCHAR(20)` | Nullable | Contact number of the user. |

### 2. `Matches` Table
Catalogs matches, cup stages, ticket pricing guidelines, and live ticket availability status.

| Column Name | Data Type | Key/Constraint | Description |
| :--- | :--- | :--- | :--- |
| `match_id` | `INT` | `PRIMARY KEY` | Unique identifier for each match. |
| `fixture` | `VARCHAR(255)` | - | Name of competing teams (e.g. Real Madrid vs Barcelona). |
| `tournament_category`| `VARCHAR(100)` | - | League/Cup category (e.g. Champions League). |
| `base_ticket_price` | `DECIMAL(10, 2)`| `CHECK (>= 0)` | Standard base ticket cost. |
| `match_status` | `VARCHAR(50)` | `CHECK` | Restricted to: `'Available'`, `'Selling Fast'`, `'Sold Out'`, `'Postponed'`. |

### 3. `Bookings` Table
Tracks ticket purchases, billing history, and stadium seat assignments.

| Column Name | Data Type | Key/Constraint | Description |
| :--- | :--- | :--- | :--- |
| `booking_id` | `INT` | `PRIMARY KEY` | Unique identifier for each booking transaction. |
| `user_id` | `INT` | `FOREIGN KEY` | References `Users(user_id)`. |
| `match_id` | `INT` | `FOREIGN KEY` | References `Matches(match_id)`. |
| `seat_number` | `VARCHAR(20)` | Nullable | Allocated seat location (e.g. A-12). |
| `payment_status` | `VARCHAR(20)` | `CHECK` (Allows `NULL`) | Restricted to: `'Pending'`, `'Confirmed'`, `'Cancelled'`, `'Refunded'`, or `NULL`. |
| `total_cost` | `DECIMAL(10, 2)`| `CHECK (>= 0)` | Final transaction amount. |

---

##  Database Implementation & Seeding

The raw PostgreSQL DDL code and corresponding mock datasets are implemented in [QUERY.sql](file:///f:/amdad-islam/next-level-batch-7/level-2_assignment-3/QUERY.sql).

### Table Creation DDL
```sql
-- Users Table
CREATE TABLE Users (
    user_id INT,
    full_name VARCHAR(100),
    email VARCHAR(100),
    role VARCHAR(50),
    phone_number VARCHAR(20),
    PRIMARY KEY (user_id),
    UNIQUE (email),
    CHECK (role IN ('Ticket Manager', 'Football Fan'))
);

-- Matches Table
CREATE TABLE Matches (
    match_id INT,
    fixture VARCHAR(255),
    tournament_category VARCHAR(100),
    base_ticket_price DECIMAL(10, 2),
    match_status VARCHAR(50),
    PRIMARY KEY (match_id),
    CHECK (base_ticket_price >= 0),
    CHECK (match_status IN ('Available', 'Selling Fast', 'Sold Out', 'Postponed'))
);

-- Bookings Table
CREATE TABLE Bookings (
    booking_id INT,
    user_id INT,
    match_id INT,
    seat_number VARCHAR(20),
    payment_status VARCHAR(20),
    total_cost DECIMAL(10, 2),
    PRIMARY KEY (booking_id),
    FOREIGN KEY (user_id) REFERENCES Users(user_id),
    FOREIGN KEY (match_id) REFERENCES Matches(match_id),
    CHECK (total_cost >= 0),
    CHECK (payment_status IS NULL OR payment_status IN ('Pending', 'Confirmed', 'Cancelled', 'Refunded'))
);
```

### Mock Seeding Snippet
The initial dataset seeds the tables to verify the database constraints:
* **Users:** Populates fans and a manager, supporting standard emails and nullable phone numbers.
* **Matches:** Seeded with standard pricing tier and status distributions (`Available`, `Selling Fast`, `Sold Out`).
* **Bookings:** Accounts for valid completed purchases, pending reservations, and entries without seat selections or payment validation (`NULL`).

---

##  Structural Features
* **Flexible Nullable Constraints:** The check constraint on `payment_status` allows `NULL` values to support cases where payment processes are incomplete.
* **Precise Monetary Storage:** Financial fields (`base_ticket_price`, `total_cost`) leverage the PostgreSQL `DECIMAL(10, 2)` type for absolute monetary precision.
* **Strong Referential Safeguards:** Restricts invalid ticket creation by enforcing strict Foreign Key references back to base entities.
