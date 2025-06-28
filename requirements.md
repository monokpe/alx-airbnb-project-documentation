## Backend Requirement Specifications for Airbnb Clone

### Feature 1: User Authentication & Management

#### 1.1. Overview

This feature handles all aspects of user identity, including registration, login, session management, and profile updates. It establishes a secure foundation for differentiating between guests, hosts, and admins using Role-Based Access Control (RBAC).

#### 1.2. Functional Requirements

- Users must be able to register with a name, email, password, and role (guest or host).
- Registered users must be able to log in using their email and password.
- The system must issue a secure JSON Web Token (JWT) upon successful login for authenticating subsequent API requests.
- Authenticated users must be able to retrieve and update their own profile information.
- Passwords must be securely stored using a strong, one-way hashing algorithm (e.g., bcrypt).

#### 1.3. API Endpoint Specifications

---

**1.3.1. User Registration**

- **Endpoint:** `POST /api/v1/auth/register`
- **Description:** Creates a new user account.
- **Request Body:**
  ```json
  {
    "name": "John Doe",
    "email": "john.doe@example.com",
    "password": "Password123!",
    "role": "guest"
  }
  ```
- **Validation Rules:**
  - `name`: Required, string, max 100 characters.
  - `email`: Required, must be a valid email format, must be unique in the `users` table.
  - `password`: Required, string, minimum 8 characters, must contain at least one uppercase letter, one lowercase letter, and one number.
  - `role`: Required, must be either `'guest'` or `'host'`.
- **Success Response (201 Created):**
  ```json
  {
    "message": "User registered successfully.",
    "user": {
      "id": "c7a8b9d0-e1f2-g3h4-i5j6-k7l8m9n0o1p2",
      "name": "John Doe",
      "email": "john.doe@example.com",
      "role": "guest"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
  ```
- **Error Responses:**
  | Status Code | Reason |
  | :--- | :--- |
  | `400 Bad Request` | Invalid input format or missing required fields. |
  | `409 Conflict` | An account with the provided email already exists. |

---

**1.3.2. User Login**

- **Endpoint:** `POST /api/v1/auth/login`
- **Description:** Authenticates a user and returns a JWT.
- **Request Body:**
  ```json
  {
    "email": "john.doe@example.com",
    "password": "Password123!"
  }
  ```
- **Validation Rules:**
  - `email`: Required, must be a valid email format.
  - `password`: Required, string.
- **Success Response (200 OK):**
  ```json
  {
    "message": "Login successful.",
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
  ```
- **Error Responses:**
  | Status Code | Reason |
  | :--- | :--- |
  | `400 Bad Request` | Missing `email` or `password`. |
  | `401 Unauthorized` | Invalid email or password. |

---

**1.3.3. Get User Profile**

- **Endpoint:** `GET /api/v1/users/me`
- **Description:** Retrieves the profile of the currently authenticated user.
- **Authentication:** Required (JWT).
- **Success Response (200 OK):**
  ```json
  {
    "id": "c7a8b9d0-e1f2-g3h4-i5j6-k7l8m9n0o1p2",
    "name": "John Doe",
    "email": "john.doe@example.com",
    "role": "guest",
    "bio": "Travel enthusiast.",
    "profilePictureUrl": "https://storage.cloud/path/to/image.jpg",
    "createdAt": "2023-10-27T10:00:00Z"
  }
  ```
- **Error Responses:**
  | Status Code | Reason |
  | :--- | :--- |
  | `401 Unauthorized` | Token is missing or invalid. |

#### 1.4. Non-Functional Requirements

- **Performance:**
  - Login and registration endpoints should have a response time of **< 250ms** under average load.
  - Profile retrieval should respond in **< 150ms**.
- **Security:**
  - Implement rate limiting on the `/login` endpoint (e.g., 5 failed attempts per minute per IP) to prevent brute-force attacks.
  - JWTs should have a reasonably short expiration (e.g., 1-24 hours) and be signed with a strong secret key stored securely as an environment variable.

### Feature 2: Property Listings Management

#### 2.1. Overview

This feature allows users with the `host` role to create, manage, and display their property listings. Guests can search and view these listings.

#### 2.2. Functional Requirements

- A user with the `host` role must be able to create a new property listing.
- A host must be able to view all of their own listings.
- A host must be able to update the details of their own listings.
- A host must be able to delete their own listings, provided there are no active future bookings.
- Any user (guest or host) must be able to view the details of a single property.

#### 2.3. API Endpoint Specifications

---

**2.3.1. Create a Property Listing**

- **Endpoint:** `POST /api/v1/properties`
- **Description:** Adds a new property listing to the platform.
- **Authentication:** Required (JWT).
- **Authorization:** User role must be `host`.
- **Request Body:**
  ```json
  {
    "title": "Cozy Downtown Apartment",
    "description": "A beautiful apartment in the heart of the city.",
    "location": "123 Main St, Anytown, USA",
    "pricePerNight": 150.0,
    "maxGuests": 4,
    "amenities": ["Wi-Fi", "Kitchen", "Air Conditioning"],
    "imageUrls": [
      "https://storage.cloud/img1.jpg",
      "https://storage.cloud/img2.jpg"
    ]
  }
  ```
- **Validation Rules:**
  - All fields are required.
  - `title`: String, 5-150 characters.
  - `description`: String, 10-1000 characters.
  - `pricePerNight`: Numeric, must be greater than 0.
  - `maxGuests`: Integer, must be greater than 0.
  - `amenities`: Array of strings.
- **Success Response (201 Created):**
  ```json
  {
    "id": "prop-a1b2c3d4-e5f6-g7h8-i9j0-k1l2m3n4o5p6",
    "hostId": "c7a8b9d0-e1f2-g3h4-i5j6-k7l8m9n0o1p2",
    "title": "Cozy Downtown Apartment",
    "pricePerNight": 150.00,
    "maxGuests": 4,
    ...
  }
  ```
- **Error Responses:**
  | Status Code | Reason |
  | :--- | :--- |
  | `400 Bad Request` | Invalid input data. |
  | `401 Unauthorized` | Token is missing or invalid. |
  | `403 Forbidden` | User's role is not `host`. |

---

**2.3.2. Delete a Property Listing**

- **Endpoint:** `DELETE /api/v1/properties/{propertyId}`
- **Description:** Deletes a property listing.
- **Authentication:** Required (JWT).
- **Authorization:** User must be the owner of the property.
- **Success Response (204 No Content):**
  - An empty response body with a 204 status code.
- **Error Responses:**
  | Status Code | Reason |
  | :--- | :--- |
  | `401 Unauthorized` | Token is missing or invalid. |
  | `403 Forbidden` | User is not the owner of the property. |
  | `404 Not Found` | Property with the given `propertyId` does not exist. |
  | `409 Conflict` | Property cannot be deleted because it has upcoming confirmed bookings. |

#### 2.4. Non-Functional Requirements

- **Performance:**
  - Property creation/update/deletion should complete in **< 300ms**.
  - Image uploads (handled via a file storage service like S3) should not block the API response. The API should respond after metadata is saved, while uploads complete asynchronously.
- **Security:**
  - Strict authorization checks must be in place for all `PUT` and `DELETE` operations to ensure hosts can only modify their own listings.

### Feature 3: Booking System

#### 3.1. Overview

This feature manages the entire lifecycle of a booking, from initiation by a guest to confirmation and cancellation. It is the transactional core of the application.

#### 3.2. Functional Requirements

- A user with the `guest` role must be able to create a booking for a specific property and date range.
- The system must prevent double-bookings for the same property on overlapping dates.
- Bookings must have a status (`pending`, `confirmed`, `cancelled`, `completed`).
- A booking is created with a `pending` status. It moves to `confirmed` after successful payment processing.
- Guests must be able to view their own past and upcoming bookings.
- Hosts must be able to view bookings for their own properties.
- Guests or hosts can cancel a booking, subject to the platform's cancellation policy.

#### 3.3. API Endpoint Specifications

---

**3.3.1. Create a Booking**

- **Endpoint:** `POST /api/v1/bookings`
- **Description:** A guest initiates a booking request for a property. This creates a `pending` booking.
- **Authentication:** Required (JWT).
- **Authorization:** User role must be `guest`.
- **Request Body:**
  ```json
  {
    "propertyId": "prop-a1b2c3d4-e5f6-g7h8-i9j0-k1l2m3n4o5p6",
    "checkInDate": "2024-08-10",
    "checkOutDate": "2024-08-15"
  }
  ```
- **Validation Rules:**
  - All fields are required.
  - `propertyId` must correspond to an existing, active property.
  - `checkInDate` and `checkOutDate` must be valid dates in `YYYY-MM-DD` format.
  - `checkInDate` must be in the future.
  - `checkOutDate` must be after `checkInDate`.
  - **Crucially, the requested date range must not overlap with any `confirmed` bookings for the given `propertyId`**.
- **Success Response (201 Created):**
  ```json
  {
    "id": "book-xyz789",
    "guestId": "c7a8b9d0-e1f2-g3h4-i5j6-k7l8m9n0o1p2",
    "propertyId": "prop-a1b2c3d4-...",
    "checkInDate": "2024-08-10",
    "checkOutDate": "2024-08-15",
    "totalPrice": 750.0,
    "status": "pending",
    "createdAt": "2023-10-27T11:00:00Z"
  }
  ```
- **Error Responses:**
  | Status Code | Reason |
  | :--- | :--- |
  | `400 Bad Request` | Invalid dates or other input. |
  | `401 Unauthorized` | Not logged in. |
  | `403 Forbidden` | A host is trying to book a property. |
  | `404 Not Found` | Property does not exist. |
  | `409 Conflict` | The property is not available for the selected dates. |

---

**3.3.2. Cancel a Booking**

- **Endpoint:** `PUT /api/v1/bookings/{bookingId}/cancel`
- **Description:** Cancels a booking. Can be initiated by the guest or the property host.
- **Authentication:** Required (JWT).
- **Authorization:** User must be the guest who made the booking OR the host who owns the property.
- **Success Response (200 OK):**
  ```json
  {
    "id": "book-xyz789",
    "status": "cancelled",
    ...
  }
  ```
- **Error Responses:**
  | Status Code | Reason |
  | :--- | :--- |
  | `401 Unauthorized` | Not logged in. |
  | `403 Forbidden` | User is not the guest or host for this booking. |
  | `404 Not Found` | Booking does not exist. |
  | `409 Conflict` | Booking cannot be cancelled (e.g., it is already completed or past the cancellation window). |

#### 3.4. Non-Functional Requirements

- **Performance:**
  - The availability check during booking creation is critical and must be highly optimized. The query should execute in **< 150ms** even with thousands of bookings in the table.
- **Reliability / Atomicity:**
  - The process of checking availability and creating a `pending` booking should be executed within a single database transaction to prevent race conditions where two users book the same dates simultaneously.
- **Data Integrity:**
  - The `status` field should be an `ENUM` type in the database to ensure only valid statuses can be saved.
