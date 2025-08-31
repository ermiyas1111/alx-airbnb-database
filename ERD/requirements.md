# Database Requirements & Entity-Relationship Diagram (ERD) Specification

## Objective
This document outlines the database schema design for the ALX Airbnb project. It defines the entities, their attributes, relationships, and constraints, serving as the blueprint for the physical database implementation.

---

## Entity Definitions

### 1. Table: `users`
Stores all registered users of the platform (guests, hosts, and admins).

| Attribute       | Type        | Constraints                  | Description                                                                 |
|-----------------|-------------|------------------------------|-----------------------------------------------------------------------------|
| `user_id`       | UUID        | PRIMARY KEY                  | Unique identifier for the user.                                             |
| `first_name`    | VARCHAR     | NOT NULL                     | User's first name.                                                          |
| `last_name`     | VARCHAR     | NOT NULL                     | User's last name.                                                           |
| `email`         | VARCHAR     | UNIQUE, NOT NULL             | User's email address, used for login.                                       |
| `password_hash` | VARCHAR     | NOT NULL                     | Securely hashed version of the user's password.                             |
| `phone_number`  | VARCHAR     | NULL                         | User's contact phone number.                                                |
| `role`          | ENUM        | NOT NULL                     | User's system role: `guest`, `host`, or `admin`.                            |
| `created_at`    | TIMESTAMP   | DEFAULT CURRENT_TIMESTAMP    | Timestamp of when the user account was created.                             |

**Rationale:** Using UUIDs enhances security and simplifies unique key generation in distributed systems. The `role` ENUM controls access and capabilities within the application.

### 2. Table: `properties`
Stores all rental listings created by hosts.

| Attribute         | Type        | Constraints                           | Description                                                              |
|-------------------|-------------|---------------------------------------|--------------------------------------------------------------------------|
| `property_id`     | UUID        | PRIMARY KEY                          | Unique identifier for the property listing.                              |
| `host_id`         | UUID        | FOREIGN KEY, NOT NULL                | References `users.user_id`. Identifies the owner of the property.        |
| `name`            | VARCHAR     | NOT NULL                             | Title of the property listing.                                           |
| `description`     | TEXT        | NOT NULL                             | Detailed description of the property.                                    |
| `location`        | VARCHAR     | NOT NULL                             | Physical address or general location of the property.                    |
| `price_per_night` | DECIMAL     | NOT NULL                             | Cost to rent the property for one night.                                 |
| `created_at`      | TIMESTAMP   | DEFAULT CURRENT_TIMESTAMP            | Timestamp of when the listing was created.                               |
| `updated_at`      | TIMESTAMP   | ON UPDATE CURRENT_TIMESTAMP          | Timestamp of the last update to the listing.                             |

**Rationale:** The `updated_at` field is crucial for tracking changes and potentially syncing with a cache. `price_per_night` is stored as DECIMAL to avoid floating-point rounding errors in financial calculations.

### 3. Table: `bookings`
Stores all reservations made by guests.

| Attribute       | Type        | Constraints                               | Description                                                                 |
|-----------------|-------------|-------------------------------------------|-----------------------------------------------------------------------------|
| `booking_id`    | UUID        | PRIMARY KEY                              | Unique identifier for the booking.                                          |
| `property_id`   | UUID        | FOREIGN KEY, NOT NULL                    | References `properties.property_id`. The booked property.                   |
| `user_id`       | UUID        | FOREIGN KEY, NOT NULL                    | References `users.user_id`. The guest who made the booking.                 |
| `start_date`    | DATE        | NOT NULL                                 | Check-in date.                                                              |
| `end_date`      | DATE        | NOT NULL                                 | Check-out date.                                                             |
| `total_price`   | DECIMAL     | NOT NULL                                 | Calculated total cost: `price_per_night * number of nights`.                |
| `status`        | ENUM        | NOT NULL                                 | Current state of the booking: `pending`, `confirmed`, or `canceled`.        |
| `created_at`    | TIMESTAMP   | DEFAULT CURRENT_TIMESTAMP                | Timestamp of when the booking was created.                                  |

**Rationale:** The `status` ENUM is vital for application workflow (e.g., preventing double-booking on `confirmed` status). `total_price` is stored to preserve the price at the time of booking, even if the property's `price_per_night` changes later.

### 4. Table: `payments`
Stores transaction records associated with bookings.

| Attribute         | Type        | Constraints                    | Description                                                         |
|-------------------|-------------|--------------------------------|---------------------------------------------------------------------|
| `payment_id`      | UUID        | PRIMARY KEY                   | Unique identifier for the payment transaction.                      |
| `booking_id`      | UUID        | FOREIGN KEY, NOT NULL         | References `bookings.booking_id`. The booking being paid for.       |
| `amount`          | DECIMAL     | NOT NULL                      | The amount charged.                                                 |
| `payment_date`    | TIMESTAMP   | DEFAULT CURRENT_TIMESTAMP     | Timestamp of when the payment was processed.                        |
| `payment_method`  | ENUM        | NOT NULL                      | Method used: `credit_card`, `paypal`, or `stripe`.                  |

**Rationale:** This table is essential for financial auditing. In a future iteration, consider adding a `status` field (e.g., `completed`, `failed`, `refunded`) for better payment lifecycle management.

### 5. Table: `reviews`
Stores ratings and reviews left by guests for properties after their stay.

| Attribute       | Type        | Constraints                               | Description                                                   |
|-----------------|-------------|-------------------------------------------|---------------------------------------------------------------|
| `review_id`     | UUID        | PRIMARY KEY                              | Unique identifier for the review.                             |
| `property_id`   | UUID        | FOREIGN KEY, NOT NULL                    | References `properties.property_id`. The reviewed property.   |
| `user_id`       | UUID        | FOREIGN KEY, NOT NULL                    | References `users.user_id`. The author of the review.         |
| `rating`        | INTEGER     | CHECK (rating >= 1 AND rating <= 5), NOT NULL | Numeric rating from 1 (lowest) to 5 (highest).          |
| `comment`       | TEXT        | NOT NULL                                 | Written feedback from the guest.                              |
| `created_at`    | TIMESTAMP   | DEFAULT CURRENT_TIMESTAMP                | Timestamp of when the review was posted.                      |

**Rationale:** The `CHECK` constraint on the application level is good, but also consider enforcing it at the database level if your DBMS (like PostgreSQL) supports it. This ensures data integrity.

### 6. Table: `messages`
Stores internal messages between users.

| Attribute         | Type        | Constraints                    | Description                                                   |
|-------------------|-------------|--------------------------------|---------------------------------------------------------------|
| `message_id`      | UUID        | PRIMARY KEY                   | Unique identifier for the message.                            |
| `sender_id`       | UUID        | FOREIGN KEY, NOT NULL         | References `users.user_id`. The user who sent the message.    |
| `recipient_id`    | UUID        | FOREIGN KEY, NOT NULL         | References `users.user_id`. The user who received the message.|
| `message_body`    | TEXT        | NOT NULL                      | The content of the message.                                   |
| `sent_at`         | TIMESTAMP   | DEFAULT CURRENT_TIMESTAMP     | Timestamp of when the message was sent.                       |

**Rationale:** This design enables a simple direct messaging system. For a more scalable solution (like conversation threads), a different structure would be needed, but this is perfect for an MVP.

---

## Relationships & Business Logic

*   **A `User`** can be a `host` for zero or more **Properties** (`1:N` relationship).
*   **A `User`** (with role `guest`) can make zero or more **Bookings** (`1:N` relationship).
*   **A `Property`** can have zero or more **Bookings** (`1:N` relationship). Application logic must check for date conflicts.
*   **A `Booking`** must have exactly one associated **Payment** record (`1:1` relationship).
*   **A `Property`** can have zero or more **Reviews** (`1:N` relationship).
*   **A `User`** can write zero or more **Reviews** (`1:N` relationship). A business rule should enforce that a user can only review properties they have stayed at.
*   **Users** can send and receive multiple **Messages** (`N:M` relationship implemented via two foreign keys in the same table).

---

## ER Diagram Visualization

The visual representation of this schema can be found in the accompanying file (`airbnb_erd.png` or `airbnb_erd.drawio`) and was generated from the following Mermaid code:

```mermaid
[erDiagram
    USER {
        UUID user_id PK
        string first_name
        string last_name
        string email UK
        string password_hash
        string phone_number
        string role
        timestamp created_at
    }

    PROPERTY {
        UUID property_id PK
        UUID host_id FK
        string name
        text description
        string location
        decimal pricepernight
        timestamp created_at
        timestamp updated_at
    }

    BOOKING {
        UUID booking_id PK
        UUID property_id FK
        UUID user_id FK
        date start_date
        date end_date
        decimal total_price
        string status
        timestamp created_at
    }

    PAYMENT {
        UUID payment_id PK
        UUID booking_id FK
        decimal amount
        timestamp payment_date
        string payment_method
    }

    REVIEW {
        UUID review_id PK
        UUID property_id FK
        UUID user_id FK
        int rating
        text comment
        timestamp created_at
    }

    MESSAGE {
        UUID message_id PK
        UUID sender_id FK
        UUID recipient_id FK
        text message_body
        timestamp sent_at
    }

    USER ||--o{ PROPERTY : hosts
    USER ||--o{ BOOKING : makes
    USER ||--o{ REVIEW : writes
    USER ||--o{ MESSAGE : sends
    USER }o--o{ MESSAGE : receives

    PROPERTY ||--o{ BOOKING : has
    PROPERTY ||--o{ REVIEW : receives

    BOOKING ||--|| PAYMENT : has
    BOOKING }|--|| USER : booked_by
    BOOKING }|--|| PROPERTY : is_forPASTE THE ENTIRE MERMAID CODE BLOCK FROM THE PREVIOUS ANSWER HERE]
