# Database Normalization to 3NF

## Objective
This document outlines the steps taken to ensure the Airbnb database schema adheres to the Third Normal Form (3NF), eliminating data redundancy and transitive dependencies.

## Normalization Steps

### 1. Initial Schema Review
The initial schema was well-structured but was analyzed for potential normalization issues:
- **Violation of 1NF:** All attributes were atomic; no repeating groups were found.
- **Violation of 2NF:** All tables had single-column primary keys (UUIDs), meaning all non-key attributes were fully functionally dependent on the primary key. No partial dependencies were found.
- **Violation of 3NF:** A potential **transitive dependency** was identified in the `properties` table.

### 2. Identified Transitive Dependency
The `location` attribute in the `properties` table was a single field containing multiple pieces of information (e.g., street, city, country). This created a transitive dependency:
`property_id` -> `location` -> `city`, `country`

This design led to:
- **Data Redundancy:** The same city and country names would be repeated for every property in that location.
- **Update Anomalies:** Changing the name of a city would require updates to all property records in that city.

### 3. Schema Adjustment for 3NF
To resolve this, the `location` attribute was normalized into a separate `locations` table.

#### New Table: `locations`
| Attribute     | Type        | Constraints               | Description                                   |
| ------------- | ----------- | ------------------------- | --------------------------------------------- |
| `location_id` | UUID        | PRIMARY KEY               | Unique identifier for the location.           |
| `street`      | VARCHAR     | NOT NULL                  | Street address.                               |
| `city`        | VARCHAR     | NOT NULL                  | City name.                                    |
| `state`       | VARCHAR     | NULL                      | State/Province/Region (if applicable).        |
| `country`     | VARCHAR     | NOT NULL                  | Country name.                                 |
| `postal_code` | VARCHAR     | NULL                      | ZIP or postal code.                           |

#### Modified Table: `properties`
The `location` attribute was replaced with a foreign key `location_id`.
| Attribute         | Type        | Constraints               | Description                                   |
| ----------------- | ----------- | ------------------------- | --------------------------------------------- |
| `property_id`     | UUID        | PRIMARY KEY               | (Remains unchanged)                           |
| `host_id`         | UUID        | FOREIGN KEY, NOT NULL     | (Remains unchanged)                           |
| `name`            | VARCHAR     | NOT NULL                  | (Remains unchanged)                           |
| `description`     | TEXT        | NOT NULL                  | (Remains unchanged)                           |
| `location_id`     | UUID        | FOREIGN KEY, NOT NULL     | **NEW:** References `locations.location_id`.  |
| `price_per_night` | DECIMAL     | NOT NULL                  | (Remains unchanged)                           |
| `created_at`      | TIMESTAMP   | DEFAULT CURRENT_TIMESTAMP | (Remains unchanged)                           |
| `updated_at`      | TIMESTAMP   | ON UPDATE CURRENT_TIMESTAMP | (Remains unchanged)                         |

### 4. Intentional Denormalization
The `total_price` attribute in the `bookings` table is a derived value (`price_per_night * number of nights`). While technically violating 3NF, it is an accepted practice for performance reasons, acting as a "calculated cache" to avoid expensive joins and calculations for frequent queries.

### 5. Conclusion
After creating the `locations` table and establishing a relationship to the `properties` table, the database schema is now in Third Normal Form (3NF). All non-key attributes depend solely on the primary key of their respective tables.

**Final 3NF-Compliant Entities:** `users`, `properties`, `locations`, `bookings`, `payments`, `reviews`, `messages`.
