    erDiagram
    LOCATIONS {
        UUID location_id PK
        string street
        string city
        string state
        string country
        string postal_code
    }

    PROPERTIES {
        UUID property_id PK
        UUID host_id FK
        UUID location_id FK
        string name
        text description
        decimal price_per_night
        timestamp created_at
        timestamp updated_at
    }

    PROPERTIES }o--|| LOCATIONS : "is located at"
