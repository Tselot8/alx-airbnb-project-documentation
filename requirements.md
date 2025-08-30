# Airbnb Clone Backend Requirements

## 1. User Authentication
### Description
This feature enables secure user registration, login, and profile management, supporting roles (guest, host, admin).

### API Endpoints
- **GET /users/**: List all users.
- **POST /users/**: Register a new user.
- **GET /users/{user_id}/**: Retrieve a specific user.
- **PUT /users/{user_id}/**: Update a specific user.
- **DELETE /users/{user_id}/**: Delete a specific user.

### Input/Output Specifications
- **POST /users/**
  - **Input**: 
    - `first_name` (VARCHAR, NOT NULL)
    - `last_name` (VARCHAR, NOT NULL)
    - `email` (VARCHAR, UNIQUE, NOT NULL)
    - `password_hash` (VARCHAR, NOT NULL)
    - `phone_number` (VARCHAR, NULL)
    - `role` (ENUM: guest, host, admin, NOT NULL)
  - **Output**: 
    - `user_id` (UUID)
    - `created_at` (TIMESTAMP)
    - Status: 201 Created
- **GET /users/{user_id}/**
  - **Input**: `user_id` (UUID)
  - **Output**: JSON object with `user_id`, `first_name`, `last_name`, `email`, `phone_number`, `role`, `created_at`
  - Status: 200 OK
- **PUT /users/{user_id}/**
  - **Input**: Updated `first_name`, `last_name`, `phone_number`, `role`
  - **Output**: Updated user object
  - Status: 200 OK
- **DELETE /users/{user_id}/**
  - **Input**: `user_id` (UUID)
  - **Output**: Status: 204 No Content

### Validation Rules
- `email` must be unique and follow a valid email format.
- `password_hash` must be securely hashed (e.g., bcrypt).
- `role` must be one of `guest`, `host`, or `admin`.
- All NOT NULL fields must be provided.

### Performance Criteria
- Response time: < 200ms for GET/POST requests.
- Throughput: Handle 1000 concurrent requests.
- Scalability: Support up to 1 million users with indexing on `user_id` and `email`.

---

## 2. Property Management
### Description
This feature allows hosts to create, update, retrieve, and delete property listings.

### API Endpoints
- **GET /properties/**: List all properties.
- **POST /properties/**: Create a new property.
- **GET /properties/{property_id}/**: Retrieve a specific property.
- **PUT /properties/{property_id}/**: Update a specific property.
- **DELETE /properties/{property_id}/**: Delete a specific property.

### Input/Output Specifications
- **POST /properties/**
  - **Input**: 
    - `host_id` (UUID, FK to User, NOT NULL)
    - `name` (VARCHAR, NOT NULL)
    - `description` (TEXT, NOT NULL)
    - `location` (VARCHAR, NOT NULL)
    - `pricepernight` (DECIMAL, NOT NULL)
  - **Output**: 
    - `property_id` (UUID)
    - `created_at` (TIMESTAMP)
    - `updated_at` (TIMESTAMP)
    - Status: 201 Created
- **GET /properties/{property_id}/**
  - **Input**: `property_id` (UUID)
  - **Output**: JSON object with `property_id`, `host_id`, `name`, `description`, `location`, `pricepernight`, `created_at`, `updated_at`
  - Status: 200 OK
- **PUT /properties/{property_id}/**
  - **Input**: Updated `name`, `description`, `location`, `pricepernight`
  - **Output**: Updated property object
  - Status: 200 OK
- **DELETE /properties/{property_id}/**
  - **Input**: `property_id` (UUID)
  - **Output**: Status: 204 No Content

### Validation Rules
- `host_id` must reference an existing `user_id` with `role = host`.
- `pricepernight` must be positive (> 0).
- All NOT NULL fields must be provided.

### Performance Criteria
- Response time: < 300ms for GET/POST requests.
- Throughput: Handle 500 concurrent requests.
- Scalability: Support 100,000 properties with indexing on `property_id` and `host_id`.

---

## 3. Booking System
### Description
This feature enables guests to create, update, and manage bookings, including status tracking.

### API Endpoints
- **GET /bookings/**: List all bookings.
- **POST /bookings/**: Create a new booking.
- **GET /bookings/{booking_id}/**: Retrieve a specific booking.
- **PUT /bookings/{booking_id}/**: Update a specific booking.
- **DELETE /bookings/{booking_id}/**: Cancel a specific booking.

### Input/Output Specifications
- **POST /bookings/**
  - **Input**: 
    - `property_id` (UUID, FK to Property, NOT NULL)
    - `user_id` (UUID, FK to User, NOT NULL)
    - `start_date` (DATE, NOT NULL)
    - `end_date` (DATE, NOT NULL)
  - **Output**: 
    - `booking_id` (UUID)
    - `total_price` (DECIMAL)
    - `status` (ENUM: pending)
    - `created_at` (TIMESTAMP)
    - Status: 201 Created
- **GET /bookings/{booking_id}/**
  - **Input**: `booking_id` (UUID)
  - **Output**: JSON object with `booking_id`, `property_id`, `user_id`, `start_date`, `end_date`, `total_price`, `status`, `created_at`
  - Status: 200 OK
- **PUT /bookings/{booking_id}/**
  - **Input**: Updated `start_date`, `end_date`, `status`
  - **Output**: Updated booking object
  - Status: 200 OK
- **DELETE /bookings/{booking_id}/**
  - **Input**: `booking_id` (UUID)
  - **Output**: Status: 204 No Content (updates `status` to canceled)

### Validation Rules
- `property_id` and `user_id` must reference existing records.
- `start_date` must be <= `end_date` and not overlap with existing bookings for the same `property_id`.
- `status` must be one of `pending`, `confirmed`, or `canceled`.
- `total_price` is calculated as `(end_date - start_date) * pricepernight` and must be positive.

### Performance Criteria
- Response time: < 400ms for GET/POST requests.
- Throughput: Handle 1000 concurrent bookings.
- Scalability: Support 500,000 bookings with indexing on `booking_id`, `property_id`, and `user_id`.