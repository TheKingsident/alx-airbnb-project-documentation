# Requirements Documentation

## 1. User Authentication

### Functional Requirements
- Allow users to register as guests or hosts.
- Enable login using email and password.
- Provide OAuth login (Google/Facebook).
- Secure user sessions using JWT.
- Allow password reset and profile updates.

### 🔧 Technical Requirements

#### API Endpoints
- `POST /api/auth/register`
- `POST /api/auth/login`
- `POST /api/auth/oauth/google`
- `POST /api/auth/oauth/facebook`
- `POST /api/auth/forgot-password`
- `PATCH /api/users/:id`

#### Input Specifications

**Register**
- `name` (string, required)
- `email` (string, required, unique, valid email format)
- `password` (string, required, min 8 chars)
- `role` (enum: guest/host, required)

**Login**
- `email` (string, required)
- `password` (string, required)

#### Output (Response)
- **201 Created (Registration)**  
    `{ token: string, user: { id, name, email, role } }`
- **200 OK (Login)**  
    `{ token: string, user: { id, name, email, role } }`
- **400 Bad Request, 401 Unauthorized, 409 Conflict** for validation/auth errors.

#### Validation Rules
- Unique email constraint.
- Secure password storage (bcrypt hashing).
- Email must be in valid format.
- Password min length: 8 characters.

#### Performance Criteria
- Token issuance within 300ms.
- Login request latency under 500ms.

---

## 2. Property Management

### Functional Requirements
- Hosts can create, update, and delete property listings.
- Listings should contain detailed info and be searchable by users.

### 🔧 Technical Requirements

#### API Endpoints
- `POST /api/properties`
- `GET /api/properties`
- `GET /api/properties/:id`
- `PUT /api/properties/:id`
- `DELETE /api/properties/:id`

#### Input Specifications

**Create/Update**
- `title` (string, required)
- `description` (string, required)
- `location` (string, required)
- `price` (decimal, required, > 0)
- `amenities` (array of strings, optional)
- `available_dates` (array of date ranges, optional)
- `images` (array of URLs or uploaded files)

#### Output (Response)
- **201 Created**  
    `{ id, title, hostId, price, location, ... }`
- **200 OK** (on GET, PUT)
- **204 No Content** (on DELETE)
- **400, 403, 404** for errors.

#### Validation Rules
- Only authenticated hosts can create/edit/delete properties.
- Required fields must not be empty.
- Price must be a positive number.

#### Performance Criteria
- Property creation/update within 800ms.
- Search queries should return results within 1s for datasets under 10,000 properties.

---

## 3. Booking System

### Functional Requirements
- Guests can book available properties for specific dates.
- The system must prevent double bookings.
- Bookings must track status (pending, confirmed, cancelled, completed).
- Hosts and guests can cancel bookings per policy.

### 🔧 Technical Requirements

#### API Endpoints
- `POST /api/bookings`
- `GET /api/bookings/:id`
- `GET /api/users/:userId/bookings`
- `PATCH /api/bookings/:id/cancel`

#### Input Specifications

**Create Booking**
- `propertyId` (UUID, required)
- `guestId` (UUID, required)
- `startDate` (ISO date, required)
- `endDate` (ISO date, required)

**Cancel Booking**
- `bookingId` (UUID)
- `reason` (string, optional)

#### Output (Response)
- **201 Created**  
    `{ id, status: "pending", startDate, endDate, guest, property }`
- **200 OK** (for cancellation or retrieval)
- **400, 409, 404** for validation, conflict, or not found errors.

#### Validation Rules
- Prevent overlapping bookings for the same property using date validation.
- Ensure `startDate` is before `endDate` and both are in the future.
- Only the guest or host involved can cancel.

#### Performance Criteria
- Booking creation latency < 700ms.
- Conflict check must scale for up to 50 concurrent bookings.
