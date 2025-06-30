# Airbnb Clone Backend - Feature Requirements

This document outlines the technical and functional requirements for key backend features of the Airbnb Clone. It includes API specifications, input/output structures, validation rules, and performance guidelines.

---

## 1. User Authentication

### Objective
Enable users to securely register, log in, and access protected resources via JWT authentication.

### Endpoints

#### POST `/api/auth/register`
- **Input:**
  ```json
  {
    "email": "user@example.com",
    "password": "yourpassword",
    "first_name": "First Name",
    "last_name": "Last Name",
    "role": "guest"
  }
  ```
- **Validations:**
  - Email must be unique and valid.
  - Password must be at least 8 characters.
  - Role must be `guest` or `host`.
  - First Name must not be empty.
  - Last Name must not be empty

- **Output:**
  ```json
  {
    "message": "Registration successful",
    "user_id": "uuid",
    "token": "jwt_token"
  }
  ```

#### POST `/api/auth/login`
- **Input:**
  ```json
  {
    "email": "user@example.com",
    "password": "yourpassword"
  }
  ```
- **Output:**
  ```json
  {
    "token": "jwt_token"
  }
  ```

### Performance
- Password hashing (e.g., bcrypt)
- Response time < 300ms

---

## 2. Property Management

### Objective
Allow hosts to manage property listings (create, update, delete, fetch).

### Endpoints

#### POST `/api/properties/`
- **Input:**
  ```json
  {
    "title": "Luxury Studio",
    "description": "Near the beach",
    "location": "Cape Town",
    "price_per_night": 75,
    "amenities": ["wifi", "kitchen"],
    "images": ["url1", "url2"]
  }
  ```
- **Validation:**
  - Required: title, location, price_per_night.
  - Price must be a number > 0.

#### GET `/api/properties/`
- **Query Params:** `location`, `min_price`, `max_price`, `guests`, `amenities`
- **Output:** List of matching properties with pagination.

#### PUT `/api/properties/:id`
- Host-only listing update.

#### DELETE `/api/properties/:id`
- Host-only listing deletion.

### Performance
- Images stored in Cloudinary or S3.
- Paginated responses.

---

## 3. Booking System

### Objective
Allow users to book properties, manage bookings, and prevent double bookings.

### Endpoints

#### POST `/api/bookings/`
- **Input:**
  ```json
  {
    "property_id": "uuid",
    "user_id": "uuid",
    "total_price": "price",
    "check_in": "2025-07-01",
    "check_out": "2025-07-05"
  }
  ```
- **Validation:**
  - Dates must be valid and not overlap.
  - Booking must be for at least 1 night.

- **Output:**
  ```json
  {
    "booking_id": "uuid",
    "status": "confirmed"
  }
  ```

#### GET `/api/bookings/`
- Get current user’s bookings.

#### DELETE `/api/bookings/:id`
- Cancel booking (with applicable rules).

### Performance
- Use DB transactions to prevent race conditions.
- Availability logic must lock inventory.

---

## 4. Payment Integration

### Objective
Process payments from guests and payouts to hosts.

### Endpoints

#### POST `/api/payments/charge`
- **Input:**
  ```json
  {
    "booking_id": "uuid",
    "payment_method": "card"
  }
  ```

- **Output:**
  ```json
  {
    "status": "paid",
    "transaction_id": "txn_123456"
  }
  ```

### Performance
- Use secure gateway (e.g., Stripe).
- Handle retries and failure callbacks.

---

## 5. Reviews & Ratings

### Objective
Allow users to leave and view reviews.

### Endpoints

#### POST `/api/reviews/`
- **Input:**
  ```json
  {
    "property_id": "uuid",
    "user_id": "user_id",
    "rating": 4,
    "comment": "Great stay!"
  }
  ```

#### GET `/api/properties/:id/reviews`
- Output: List of reviews per property.

### Rules
- Only users with a completed booking can post a review.

---

## 6. Admin Dashboard

### Objective
Allow admins to manage users, properties, bookings, and reports.

### Endpoints
- GET `/api/admin/users/`
- DELETE `/api/admin/properties/:id`
- GET `/api/admin/bookings/`

### Performance
- Role-based access control required.

---

## Common Standards

- All routes (except login/register) require JWT.
- Status Codes: 200 (OK), 201 (Created), 400 (Bad Request), 401/403 (Unauthorized), 404 (Not Found)
