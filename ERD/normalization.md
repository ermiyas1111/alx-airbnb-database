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
```sql
CREATE TABLE locations (
    location_id UUID PRIMARY KEY,
    street VARCHAR(255) NOT NULL,
    city VARCHAR(255) NOT NULL,
    state VARCHAR(255),
    country VARCHAR(255) NOT NULL,
    postal_code VARCHAR(50)
);
