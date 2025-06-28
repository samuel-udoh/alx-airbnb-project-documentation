## Technical and Functional Requirements Document

This document outlines the functional and technical requirements for the backend service of the Airbnb Clone project. It is designed to guide the development, testing, and deployment of the platform.

### 1. User Management

#### Functional Requirements

- **FR1.1 - User Registration:** Users must be able to create a new account by providing an email, password, first name, and last name. They should select a role upon registration (`Guest` or `Host`).
- **FR1.2 - User Login:** Registered users must be able to log in using their email and password.
- **FR1.3 - OAuth Authentication:** Users must have the option to register and log in using third-party providers (e.g., Google, Facebook).
- **FR1.4 - Profile Management:** Authenticated users must be able to view and update their profile information, including their name, contact info, and profile photo.
- **FR1.5 - Secure Logout:** Users must be able to log out, which will invalidate their current session token.

#### Technical Requirements

- **TR1.1 - Database Schema:** The `Users` table will store user data. Key fields include: `id`, `email`, `password_hash`, `first_name`, `last_name`, `role` (ENUM: 'guest', 'host', 'admin'), `created_at`, `updated_at`.
- **TR1.2 - Authentication:**
  - Implement session management using **JSON Web Tokens (JWT)**. A signed JWT will be returned upon successful login.
  - The JWT payload will include `user_id` and `role` to facilitate role-based access control.
- **TR1.3 - Security:** Passwords must be hashed using a strong, salted algorithm like **bcrypt**. Plain-text passwords must never be stored.
- **TR1.4 - API Endpoints:**
  - `POST /api/auth/register` - Create a new user.
  - `POST /api/auth/login` - Authenticate a user and return a JWT.
  - `POST /api/auth/oauth/google` - Handle Google OAuth callback.
  - `PUT /api/users/me` - Allow a user to update their own profile.

### 2. Property Listings Management

#### Functional Requirements

- **FR2.1 - Add Listing:** A user with the `Host` role must be able to create a new property listing by providing a title, description, location (address), price per night, amenities, and number of guests.
- **FR2.2 - Upload Photos:** Hosts must be able to upload multiple photos for their property listings.
- **FR2.3 - Edit/Delete Listing:** Hosts must be able to update the details of their existing listings or remove them entirely.
- **FR2.4 - View Listings:** All users (including unauthenticated ones) can view property listings.

#### Technical Requirements

- **TR2.1 - Database Schema:** The `Properties` table will store listing data. Key fields: `id`, `host_id` (Foreign Key to `Users`), `title`, `description`, `address`, `price_per_night`, `amenities` (JSON or Array), `created_at`. A separate `Property_Images` table will link images to properties.
- **TR2.2 - Authorization (RBAC):** Access to modification endpoints must be restricted. Only the `Host` who owns the property or an `Admin` can edit or delete it.
- **TR2.3 - API Endpoints:**
  - `POST /api/properties` - (Host) Create a new property listing.
  - `PUT /api/properties/:id` - (Host) Update a specific property.
  - `DELETE /api/properties/:id` - (Host) Delete a specific property.
  - `GET /api/properties/:id` - Get public details for a single property.

### 3. Search and Filtering

#### Functional Requirements

- **FR3.1 - Search Functionality:** Users must be able to search for properties based on location (e.g., city, state, country).
- **FR3.2 - Filtering:** Users must be able to refine search results by price range, number of guests, and specific amenities (e.g., Wi-Fi, pool).
- **FR3.3 - Pagination:** For searches that return a large number of results, the data must be paginated to ensure fast load times.

#### Technical Requirements

- **TR3.1 - API Endpoint:** A single endpoint will handle all search and filter logic:
  - `GET /api/properties?location=...&min_price=...&guests=...&page=...&limit=...`
- **TR3.2 - Database Optimization:**
  - Database indexes must be created on all searchable and filterable columns (`location`, `price_per_night`, `max_guests`) to ensure fast query performance.
  - For location-based search, consider using geospatial indexes (e.g., PostGIS for PostgreSQL).
- **TR3.3 - Performance Optimization:** Implement a caching layer (e.g., **Redis**) to store results for common search queries, reducing database load and improving response times.

### 4. Booking Management

#### Functional Requirements

- **FR4.1 - Create Booking:** A `Guest` must be able to book a property for a specified date range.
- **FR4.2 - Prevent Double Booking:** The system must prevent a property from being booked for dates that overlap with an existing confirmed booking.
- **FR4.3 - Booking Cancellation:** Both the `Guest` who made the booking and the `Host` who owns the property must be able to cancel a booking. Cancellation policies may apply.
- **FR4.4 - Track Booking Status:** The system must track the status of each booking (`pending`, `confirmed`, `cancelled`, `completed`).

#### Technical Requirements

- **TR4.1 - Database Schema:** The `Bookings` table will track all reservations. Key fields: `id`, `guest_id`, `property_id`, `start_date`, `end_date`, `total_price`, `status` (ENUM), `created_at`.
- **TR4.2 - Concurrency Control:** The booking creation logic must be atomic. Use database transactions to check for date availability and create the booking in a single, indivisible operation to prevent race conditions.
- **TR4.3 - API Endpoints:**
  - `POST /api/bookings` - (Guest) Create a new booking.
  - `PUT /api/bookings/:id/cancel` - (Guest/Host) Cancel a booking.
  - `GET /api/bookings/my-bookings` - Get all bookings for the authenticated user.

### 5. Payment Integration

#### Functional Requirements

- **FR5.1 - Upfront Payment:** Guests must be able to pay for their booking in full using a secure payment gateway (e.g., Stripe, PayPal).
- **FR5.2 - Host Payouts:** The system should facilitate automatic payouts to the host after a booking is successfully completed.
- **FR5.3 - Multi-Currency Support:** The system should be able to process payments in multiple currencies.

#### Technical Requirements

- **TR5.1 - Database Schema:** The `Payments` table will log transactions. Key fields: `id`, `booking_id`, `amount`, `currency`, `status` (ENUM: 'succeeded', 'failed'), `transaction_id` (from gateway), `created_at`.
- **TR5.2 - Payment Gateway Integration:** Integrate with a third-party payment provider API (e.g., Stripe). All payment processing will be handled by the provider to ensure PCI compliance. The backend will only store transaction IDs, not sensitive credit card information.
- **TR5.3 - Security:** All communication with the payment gateway must be over HTTPS. Use webhooks from the payment gateway to securely and reliably update payment statuses.
- **TR5.4 - API Endpoints:**
  - `POST /api/payments/create-checkout-session` - (Guest) Initiate the payment process for a booking.

### 6. Reviews and Ratings

#### Functional Requirements

- **FR6.1 - Leave Review:** After a booking is completed, a `Guest` must be able to leave a text review and a star rating (1-5) for the property.
- **FR6.2 - Respond to Review:** A `Host` must be able to post a public response to a review left on their property.
- **FR6.3 - Prevent Abuse:** Only guests who have completed a booking for a specific property can leave a review for it.

#### Technical Requirements

- **TR6.1 - Database Schema:** The `Reviews` table will store all reviews. Key fields: `id`, `booking_id`, `guest_id`, `property_id`, `rating`, `comment`, `host_response`, `created_at`.
- **TR6.2 - Authorization:** The business logic will check that the `booking_id` corresponds to the authenticated `Guest` and has a `completed` status before allowing a review to be posted.
- **TR6.3 - API Endpoints:**
  - `POST /api/reviews` - (Guest) Submit a review for a completed booking.
  - `PUT /api/reviews/:id/respond` - (Host) Add a response to a review.

### 7. Notifications System

#### Functional Requirements

- **FR7.1 - Event-Driven Notifications:** Users must receive notifications for key events.
- **FR7.2 - Notification Events:** Events include booking confirmations, cancellations, payment status updates, and new messages.
- **FR7.3 - Delivery Channels:** Notifications should be available both in-app and via email.

#### Technical Requirements

- **TR7.1 - Architecture:** Implement an event-driven or observer pattern. When a significant event occurs (e.g., booking confirmed), a "Notification" event is published.
- **TR7.2 - Service Integration:** A notification service will listen for these events and dispatch notifications via the appropriate channels (e.g., using an email service like SendGrid and a WebSocket service for in-app notifications).
- **TR7.3 - Database Schema:** A `Notifications` table may be used to store in-app notifications. Key fields: `id`, `user_id`, `message`, `is_read`, `created_at`.

### 8. Admin Dashboard

#### Functional Requirements

- **FR8.1 - User Management:** An `Admin` must be able to view, deactivate, or delete user accounts.
- **FR8.2 - Listings Management:** An `Admin` must be able to view, edit, or remove any property listing on the platform.
- **FR8.3 - Bookings Management:** An `Admin` must be able to view and manage all bookings in the system.
- **FR8.4 - Payments Monitoring:** An `Admin` must be able to view a log of all payment transactions.

#### Technical Requirements

- **TR8.1 - Authorization (RBAC):** A separate set of API endpoints will be created for admin functionalities. These endpoints will be protected by a middleware that checks if the user's JWT contains the `admin` role.
- **TR8.2 - API Endpoints:**
  - `GET /api/admin/users`
  - `DELETE /api/admin/users/:id`
  - `GET /api/admin/properties`
  - `PUT /api/admin/properties/:id`
  - `GET /api/admin/bookings`
  - `GET /api/admin/payments`
