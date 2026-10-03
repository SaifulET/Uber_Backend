# Uber Backend API Documentation

This document lists all HTTP APIs and Socket.IO events exposed by the backend.

Base URL:

```txt
https://your-api-domain.com
```

Protected endpoints require:

```http
Authorization: Bearer <access_token>
Content-Type: application/json
```

Upload endpoints use `multipart/form-data` with file field name:

```txt
image
```

Standard success response is usually:

```json
{
  "success": true,
  "message": "Operation message",
  "data": {}
}
```

Standard error response is usually one of:

```json
{
  "success": false,
  "message": "Error message",
  "error": null
}
```

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Error message"
  }
}
```

## Mounted Routers

| Base path | Module |
| --- | --- |
| `/auth` | User auth and profile |
| `/driverOnboarding` | Driver onboarding write/upload APIs |
| `/driverOnboardingRead` | Driver onboarding read APIs |
| `/riderGetRide` | Rider ride, payment, review, support APIs |
| `/driverHome` | Driver home, location, ride, earning APIs |
| `/chat` | Trip chat HTTP APIs |
| `/admin/auth` | Admin auth |
| `/admin/config` | Fare/payment configuration |
| `/admin/dashboard` | Admin dashboard |
| `/admin/drivers` | Admin driver management |
| `/admin/riders` | Admin rider management |
| `/admin/customer-support` | Admin tickets/reports |
| `/legal-content` | Public legal content |
| `/admin/legal-content` | Admin legal content |

## Auth APIs

### Register

```http
POST /auth/register
```

Public. Creates a user and sends email verification OTP.

Body:

```json
{
  "name": "Saif",
  "phone": "5551234567",
  "email": "saif@example.com",
  "password": "secret123"
}
```

Notes:

- `email` and `password` are required.
- `password` must be at least 6 characters.
- Phone is normalized as a US number if supplied.

Returns created user and message.

### Login

```http
POST /auth/login
```

Public. Logs in active user.

Body:

```json
{
  "email": "saif@example.com",
  "password": "secret123"
}
```

Returns:

```json
{
  "accessToken": "jwt",
  "refreshToken": "jwt",
  "user": {}
}
```

### Google Login

```http
POST /auth/google
```

Public. Logs in or creates a user using Google ID token.

Body:

```json
{
  "idToken": "google-id-token",
  "role": "rider"
}
```

`role` is optional and can be `rider` or `driver`.

### Edit Role

```http
PATCH /auth/editRole
```

Public in current code. Updates user role by email.

Body:

```json
{
  "email": "saif@example.com",
  "role": "driver"
}
```

Allowed roles: `rider`, `driver`, `admin`.

### Send Email Verification OTP

```http
POST /auth/send-verification
```

Public.

Body:

```json
{
  "email": "saif@example.com"
}
```

Returns message that verification code was sent.

### Verify Email OTP

```http
POST /auth/verify-email
```

Public. Verifies email and issues tokens.

Body:

```json
{
  "email": "saif@example.com",
  "otp": "1234"
}
```

Returns user, `accessToken`, and `refreshToken`.

### Forgot Password

```http
POST /auth/forgot-password
```

Public. Sends reset OTP if account exists.

Body:

```json
{
  "email": "saif@example.com"
}
```

### Verify Reset OTP

```http
POST /auth/verify-reset-otp
```

Public. Consumes reset OTP and returns a short-lived `resetToken`.

Body:

```json
{
  "email": "saif@example.com",
  "otp": "1234"
}
```

### Reset Password

```http
POST /auth/reset-password
```

Public.

Body:

```json
{
  "resetToken": "reset-token",
  "newPassword": "newSecret123",
  "confirmPassword": "newSecret123"
}
```

### Refresh Access Token

```http
POST /auth/refresh
```

Public.

Body:

```json
{
  "refreshToken": "refresh-jwt"
}
```

Returns a new `accessToken`.

### Logout

```http
POST /auth/logout
```

Public.

Body:

```json
{
  "refreshToken": "refresh-jwt"
}
```

### Get Me

```http
GET /auth/me
```

Protected. Returns authenticated user.

### Change Password

```http
PATCH /auth/change-password
```

Protected.

Body:

```json
{
  "currentPassword": "oldSecret123",
  "newPassword": "newSecret123"
}
```

### Delete Account

```http
DELETE /auth/delete-account
```

Protected. Soft-deletes account, suspends user, and revokes sessions.

### Save Profile Image

```http
PATCH /auth/image-save
```

Protected. Uploads profile image to S3.

Form data:

```txt
image=<file>
```

For drivers, this also syncs the `profile_photo` onboarding document.

### Update Profile Info

```http
PATCH /auth/profileInfo-update
```

Protected. Updates profile fields accepted by auth service.

Typical body:

```json
{
  "name": "Saif Ahmed",
  "phone": "+15551234567",
  "address": {
    "line1": "123 Main St",
    "city": "New York",
    "state": "NY",
    "postalCode": "10001",
    "country": "US"
  }
}
```

## Driver Onboarding APIs

All `/driverOnboarding` APIs are protected and require the authenticated user to be a driver.

### Get Onboarding Status

```http
GET /driverOnboarding/status
```

Returns driver onboarding status, document summary, required actions, and profile state.

### Update Onboarding Status

```http
PATCH /driverOnboarding/status
```

Updates onboarding status fields handled by the service.

Body example:

```json
{
  "status": "pending"
}
```

### Save Vehicle

```http
POST /driverOnboarding/vehicle
```

Body:

```json
{
  "brand": "Toyota",
  "model": "Prius",
  "year": 2022,
  "type": "car",
  "tier": "regular",
  "seats": 4,
  "licensePlate": "DHA-1234"
}
```

Required: `brand`, `model`, `type`, `licensePlate`.

Allowed `type`: `car`, `suv`, `van`.

Allowed `tier`: `regular`, `premium`.

### Upload Profile Photo

```http
POST /driverOnboarding/profile-photo
```

Form data:

```txt
image=<file>
```

### Upload License Front

```http
POST /driverOnboarding/license-front
```

Form data:

```txt
image=<file>
```

### Upload License Back

```http
POST /driverOnboarding/license-back
```

Form data:

```txt
image=<file>
```

### Upload Vehicle Registration

```http
POST /driverOnboarding/vehicle-registration
```

Form data:

```txt
image=<file>
```

### Upload Vehicle Insurance

```http
POST /driverOnboarding/vehicle-insurance
```

Form data:

```txt
image=<file>
```

### Create Stripe Onboarding Link

```http
POST /driverOnboarding/stripe/onboarding-link
```

Body:

```json
{
  "returnUrl": "https://your-frontend.com/driver/onboarding/stripe-return"
}
```

Returns Stripe account onboarding URL/redirect information.

### Connect Stripe

```http
POST /driverOnboarding/connect-stripe
```

Body:

```json
{
  "stripeAccountId": "acct_xxx"
}
```

### Review Driver Onboarding

```http
GET /driverOnboarding/review
```

Returns final onboarding review state.

## Driver Onboarding Read APIs

All `/driverOnboardingRead` APIs are protected.

| Method | Endpoint | Description |
| --- | --- | --- |
| `GET` | `/driverOnboardingRead/summary` | Full onboarding read summary |
| `GET` | `/driverOnboardingRead/profile-image` | Profile image document |
| `GET` | `/driverOnboardingRead/license-photos` | License front/back |
| `GET` | `/driverOnboardingRead/vehicle-registration` | Vehicle registration document |
| `GET` | `/driverOnboardingRead/vehicle-insurance` | Insurance document |
| `GET` | `/driverOnboardingRead/stripe` | Stripe account info |
| `GET` | `/driverOnboardingRead/stripe/approval` | Stripe approval/connect status |
| `GET` | `/driverOnboardingRead/vehicle-info` | Vehicle info |

## Rider Ride APIs

All `/riderGetRide` APIs are protected.

For deeper map-specific integration details, see [MAP_RELATED_API.md](C:/Saiful/Uber_Backend/MAP_RELATED_API.md).

### Rider Home

```http
GET /riderGetRide/home
```

Returns rider profile summary, payment method state, recent places, active request, and active trip.

### Recent Places

```http
GET /riderGetRide/recent-places
```

Returns saved recent pickup/dropoff places.

### Ride Options

```http
POST /riderGetRide/ride-options
```

Body:

```json
{
  "estimatedMiles": 6.2,
  "estimatedMinutes": 24
}
```

Returns fare estimates for car/SUV/van options.

### Nearby Online Drivers

```http
POST /riderGetRide/nearby-drivers
```

Body:

```json
{
  "lat": 23.7806,
  "lng": 90.4193
}
```

Returns nearby available drivers within 10 km.

### Get Payment Method Status

```http
GET /riderGetRide/payment-method
```

Returns Stripe customer/default payment method status and publishable key.

### Create Payment Setup Intent

```http
POST /riderGetRide/payment-method/setup-intent
```

Creates Stripe setup intent for saving a card.

Returns:

```json
{
  "clientSecret": "seti_xxx_secret_xxx",
  "setupIntentId": "seti_xxx",
  "customerId": "cus_xxx",
  "publishableKey": "pk_xxx"
}
```

### Save Payment Method

```http
POST /riderGetRide/payment-method/save
```

Body:

```json
{
  "setupIntentId": "seti_xxx"
}
```

Saves the Stripe payment method from the setup intent as default.

### Create Ride Request

```http
POST /riderGetRide/ride-request
```

Body:

```json
{
  "pickup": {
    "address": "Gulshan 1, Dhaka",
    "lat": 23.7806,
    "lng": 90.4193,
    "label": "Pickup"
  },
  "dropoff": {
    "address": "Airport, Dhaka",
    "lat": 23.8433,
    "lng": 90.3978,
    "label": "Airport"
  },
  "schedule": {
    "kind": "now",
    "pickupAt": null
  },
  "preference": {
    "vehicleType": "car",
    "tier": "regular",
    "seats": 4
  },
  "estimatedMiles": 12,
  "estimatedMinutes": 42
}
```

Creates a searching ride request and emits realtime events.

### Get Active Rider State

```http
GET /riderGetRide/active
```

Returns active request and active trip.

### Rider Trips

```http
GET /riderGetRide/trips
```

Returns rider trip history.

### Change Ride Request Destination

```http
PATCH /riderGetRide/ride-request/:requestId/change-destination
```

Body:

```json
{
  "pickup": {
    "address": "New pickup",
    "lat": 23.7937,
    "lng": 90.4001
  },
  "dropoff": {
    "address": "New dropoff",
    "lat": 23.7461,
    "lng": 90.3742
  },
  "estimatedMiles": 8,
  "estimatedMinutes": 30
}
```

At least one of `pickup` or `dropoff` is required.

### Check Ride Request Fare

```http
POST /riderGetRide/ride-request/:requestId/check-fare
```

Body:

```json
{
  "dropoff": {
    "address": "Airport, Dhaka",
    "lat": 23.8433,
    "lng": 90.3978
  },
  "estimatedMiles": 12,
  "estimatedMinutes": 42
}
```

Returns current quote and checked quote without saving.

### Cancel Ride Request

```http
PATCH /riderGetRide/ride-request/:requestId/cancel
```

Cancels active ride request.

### Cancel Rider Trip

```http
PATCH /riderGetRide/trip/:tripId/cancel
```

Body:

```json
{
  "reason": "Changed my mind"
}
```

May charge cancellation fee depending on trip status.

### Check Fare After Trip Destination Change

```http
POST /riderGetRide/trip/:tripId/check-fare
```

Body:

```json
{
  "dropoff": {
    "address": "New destination",
    "lat": 23.8433,
    "lng": 90.3978
  },
  "estimatedMiles": 12,
  "estimatedMinutes": 42
}
```

### Change Active Trip Destination

```http
PATCH /riderGetRide/trip/:tripId/change-destination
```

Body:

```json
{
  "dropoff": {
    "address": "New destination",
    "lat": 23.8433,
    "lng": 90.3978
  },
  "estimatedMiles": 12,
  "estimatedMinutes": 42
}
```

Emits `trip:destination-changed` to rider and driver.

### Get Driver Profile For Trip

```http
GET /riderGetRide/trip/:tripId/driver-profile
```

Returns driver profile, vehicle, reviews, and platform duration for the trip.

### Get Rider Reviews

```http
GET /riderGetRide/my-reviews
```

Returns reviews received by the rider.

### Get Driver Reviews

```http
GET /riderGetRide/drivers/:driverId/reviews
```

Returns public reviews for a driver.

### Get Trip Details

```http
GET /riderGetRide/trip/:tripId/details
```

Returns rider trip detail, driver, vehicle, pricing, and review state.

### Get Trip Payment Summary

```http
GET /riderGetRide/trip/:tripId/payment-summary
```

Returns payment breakdown for a completed trip.

### Create Trip Payment Intent

```http
POST /riderGetRide/trip/:tripId/payment-intent
```

Body:

```json
{
  "savePaymentMethod": true
}
```

Returns Stripe payment intent `clientSecret`, amount breakdown, and publishable key.

### Verify Trip Payment

```http
POST /riderGetRide/trip/:tripId/payment-verify
```

Body:

```json
{
  "paymentIntentId": "pi_xxx",
  "savePaymentMethod": true
}
```

Verifies Stripe payment and marks trip/payment record.

### Submit Driver Rating

```http
POST /riderGetRide/trip/:tripId/rating
POST /riderGetRide/trip/:tripId/driver-review
```

Both routes call the same controller.

Body:

```json
{
  "stars": 5,
  "comment": "Great ride"
}
```

### Create Rider Support Ticket

```http
POST /riderGetRide/support-ticket
```

Body:

```json
{
  "title": "Payment issue",
  "message": "I was charged twice",
  "againstUserId": "optionalUserId",
  "tripId": "optionalTripId"
}
```

## Driver Home APIs

All `/driverHome` APIs are protected and require a driver.

### Driver Home

```http
GET /driverHome/home
```

Returns driver profile, active request/trip, document state, and required actions.

### Go Online

```http
PATCH /driverHome/go-online
```

Body:

```json
{
  "lat": 23.7806,
  "lng": 90.4193
}
```

Marks driver online. Location is required if no previous location exists.

### Go Offline

```http
PATCH /driverHome/go-offline
```

Marks driver offline. Fails during active trip.

### Update Location

```http
PATCH /driverHome/location
```

Body:

```json
{
  "lat": 23.781,
  "lng": 90.418
}
```

Updates driver location and emits `trip:driver-location` to rider during active trips.

### Next Ride Request

```http
GET /driverHome/ride-requests/next
```

Returns oldest nearby request within 10 km.

### Nearby Ride Requests

```http
GET /driverHome/ride-requests/nearby
```

Returns nearby request queue.

### Accept Ride Request

```http
PATCH /driverHome/ride-requests/:requestId/accept
```

Creates a trip, marks driver busy, and emits realtime match events.

### Active Trip

```http
GET /driverHome/trip/active
```

Returns active driver trip.

### Driver Trips

```http
GET /driverHome/trips
```

Returns driver trip history.

### Today Earnings

```http
GET /driverHome/earnings-today
```

Returns current UTC-day earnings summary.

### Earnings Summary

```http
GET /driverHome/earnings-summary
```

Query params:

```txt
period=day|week|month|year
year=2026
month=10
week=1
```

Returns earnings analytics and available periods.

### Arrived At Pickup

```http
PATCH /driverHome/trip/:tripId/arrived-pickup
```

Moves trip from `accepted` to `driver_arrived`.

### Verify OTP

```http
PATCH /driverHome/trip/:tripId/verify-otp
```

Body:

```json
{
  "otp": "123456"
}
```

Verifies rider OTP and starts trip.

### Start Trip

```http
PATCH /driverHome/trip/:tripId/start
```

Starts a trip that is ready to start. Prefer `verify-otp` in normal app flow.

### Get Rider Profile For Trip

```http
GET /driverHome/trip/:tripId/rider-profile
```

Returns rider profile and reviews.

### Driver Reviews

```http
GET /driverHome/my-reviews
```

Returns reviews received by the driver.

### Rider Reviews

```http
GET /driverHome/riders/:riderId/reviews
```

Returns reviews for a rider.

### Submit Rider Rating

```http
POST /driverHome/trip/:tripId/rider-review
```

Body:

```json
{
  "stars": 5,
  "comment": "Polite rider"
}
```

### Complete Trip

```http
PATCH /driverHome/trip/:tripId/complete
```

Body:

```json
{
  "distanceMiles": 12.4
}
```

`distanceMiles` is optional. Completes trip, calculates final fare, attempts auto-charge, and emits `trip:completed`.

### Cancel Trip

```http
PATCH /driverHome/trip/:tripId/cancel
```

Body:

```json
{
  "reason": "Vehicle issue"
}
```

### Trip Completion Summary

```http
GET /driverHome/trip/:tripId/summary
```

Returns completed/cancelled trip summary, payment summary, and rating state.

### Create Driver Support Ticket

```http
POST /driverHome/support-ticket
```

Body:

```json
{
  "title": "Trip issue",
  "message": "Rider did not show up",
  "againstUserId": "optionalUserId",
  "tripId": "optionalTripId"
}
```

## Chat HTTP APIs

All `/chat` APIs are protected. The user must be a participant in the trip.

### Get Chat Header

```http
GET /chat/trips/:tripId/chat/header
```

Returns trip chat header and other participant summary.

### Get Chat Messages

```http
GET /chat/trips/:tripId/chat/messages
```

Returns trip messages.

### Send Chat Message

```http
POST /chat/trips/:tripId/chat/messages
```

Body:

```json
{
  "text": "I am waiting at the gate"
}
```

Creates message and emits chat realtime event.

### Mark Chat Seen

```http
PATCH /chat/trips/:tripId/chat/seen
```

Marks messages as seen.

### Get Unread Count

```http
GET /chat/trips/:tripId/chat/unread-count
```

Returns unread count for authenticated user.

## Admin Auth APIs

### Admin Sign In

```http
POST /admin/auth/signin
```

Body:

```json
{
  "email": "admin@example.com",
  "password": "secret123"
}
```

Returns admin user, access token, and refresh token.

### Admin Forgot Password

```http
POST /admin/auth/forgot-password
```

Body:

```json
{
  "email": "admin@example.com"
}
```

### Admin Verify Code

```http
POST /admin/auth/verify-code
```

Body:

```json
{
  "email": "admin@example.com",
  "otp": "1234"
}
```

Returns reset token.

### Admin Set New Password

```http
POST /admin/auth/set-new-password
```

Body:

```json
{
  "resetToken": "reset-token",
  "newPassword": "newSecret123",
  "confirmPassword": "newSecret123"
}
```

### Admin Refresh

```http
POST /admin/auth/refresh
```

Body:

```json
{
  "refreshToken": "refresh-jwt"
}
```

### Admin Logout

```http
POST /admin/auth/logout
```

Body:

```json
{
  "refreshToken": "refresh-jwt"
}
```

### Admin Change Name

```http
PATCH /admin/auth/change-name
```

Protected.

Body:

```json
{
  "name": "Admin Name"
}
```

### Admin Change Password

```http
PATCH /admin/auth/change-password
```

Protected.

Body:

```json
{
  "currentPassword": "oldSecret123",
  "newPassword": "newSecret123"
}
```

## Admin Config APIs

All `/admin/config` APIs are protected and require admin role.

### Get Payment Share

```http
GET /admin/config/payment-share
```

Returns:

```json
{
  "driverGets": 80,
  "received": 20
}
```

### Get Active Fare Config

```http
GET /admin/config
```

### Create Fare Config

```http
POST /admin/config
```

Body:

```json
{
  "currency": "USD",
  "pricePerMile": 2.5,
  "pricePerMinute": 0.25,
  "baseFare": {
    "car_regular": 2.5,
    "car_premium": 5,
    "suv_compact_regular": 4,
    "suv_compact_premium": 7,
    "suv_full_regular": 6,
    "suv_full_premium": 10,
    "van_compact_regular": 5,
    "van_compact_premium": 8,
    "van_full_regular": 7,
    "van_full_premium": 12
  }
}
```

### Update Fare Config

```http
PATCH /admin/config
```

Body can include any fields from create plus `driverSharePercent`:

```json
{
  "currency": "USD",
  "pricePerMile": 2.75,
  "pricePerMinute": 0.3,
  "driverSharePercent": 80,
  "baseFare": {
    "car_regular": 3
  }
}
```

## Admin Dashboard APIs

All `/admin/dashboard` APIs are protected.

### Overview

```http
GET /admin/dashboard/overview
```

Query params may include date/year filters handled by service.

Returns high-level counts, revenue, riders, drivers, and recent metrics.

### Analytics

```http
GET /admin/dashboard/analytics
```

Query params may include year/period filters handled by service.

Returns analytics chart data.

## Admin Driver Management APIs

All `/admin/drivers` APIs are protected and require admin role.

### List Drivers

```http
GET /admin/drivers
```

Common query params:

```txt
search=saif
status=active
page=1
limit=10
```

Returns paginated driver list.

### Get Driver Detail

```http
GET /admin/drivers/:driverId
```

### Get Driver Documents

```http
GET /admin/drivers/:driverId/documents
```

### Get Driver Document Detail

```http
GET /admin/drivers/:driverId/documents/:type
```

Common document types include:

```txt
profile_photo
license_front
license_back
vehicle_registration
vehicle_insurance
vehicle
stripe
```

### Review Driver Document

```http
PATCH /admin/drivers/:driverId/documents/:type/review
```

Body:

```json
{
  "status": "approved",
  "rejectionReason": ""
}
```

Allowed status mapping accepts review-style values used by the service such as `approved` and `rejected`.

### Get Driver History

```http
GET /admin/drivers/:driverId/history
```

Returns driver trip history.

### Get Driver Trip Detail

```http
GET /admin/drivers/:driverId/history/:tripId
```

### Get Driver Reports

```http
GET /admin/drivers/:driverId/reports
```

### Update Driver Account Status

```http
PATCH /admin/drivers/:driverId/account-status
```

Body:

```json
{
  "status": "active"
}
```

Common statuses: `active`, `suspended`.

### Delete Driver

```http
DELETE /admin/drivers/:driverId
```

Soft-deletes driver.

### Restore Driver

```http
PATCH /admin/drivers/:driverId/restore
```

Restores soft-deleted driver.

### Hard Delete Driver

```http
DELETE /admin/drivers/:driverId/hard-delete
```

Permanently deletes driver data supported by service.

## Admin Rider Management APIs

All `/admin/riders` APIs are protected and require admin role.

### List Riders

```http
GET /admin/riders
```

Common query params:

```txt
search=saif
status=active
page=1
limit=10
```

### List All Rider Payments

```http
GET /admin/riders/payments
```

Common query params:

```txt
page=1
limit=10
status=succeeded
```

### Get Rider Detail

```http
GET /admin/riders/:riderId
```

### Get Rider Payments

```http
GET /admin/riders/:riderId/payments
```

### Get Rider History

```http
GET /admin/riders/:riderId/history
```

### Get Rider Trip Detail

```http
GET /admin/riders/:riderId/history/:tripId
```

### Get Rider Reports

```http
GET /admin/riders/:riderId/reports
```

### Update Rider Account Status

```http
PATCH /admin/riders/:riderId/account-status
```

Body:

```json
{
  "status": "active"
}
```

Common statuses: `active`, `suspended`.

### Delete Rider

```http
DELETE /admin/riders/:riderId
```

Soft-deletes rider.

### Restore Rider

```http
PATCH /admin/riders/:riderId/restore
```

### Hard Delete Rider

```http
DELETE /admin/riders/:riderId/hard-delete
```

## Admin Customer Support APIs

All `/admin/customer-support` APIs are protected and require admin role.

### List Support Items

```http
GET /admin/customer-support
```

Returns support tickets and reports. Common query params:

```txt
entryType=ticket|report
status=pending|resolved|reviewed|received
search=keyword
page=1
limit=10
```

### Get Support Item Detail

```http
GET /admin/customer-support/:entryId
```

Finds either support ticket or report by id.

### Take Support Action

```http
PATCH /admin/customer-support/:entryId/action
```

Body for sending message:

```json
{
  "actionType": "send_message",
  "message": "We are reviewing this issue.",
  "targetUserId": "optionalRelatedUserId"
}
```

Body for resolving:

```json
{
  "actionType": "mark_resolved",
  "message": "Resolved by admin"
}
```

Allowed `actionType`: `send_message`, `mark_resolved`.

### Delete Support Item

```http
DELETE /admin/customer-support/:entryId
```

Deletes support ticket or report.

## Legal Content APIs

Supported content types:

```txt
terms-and-conditions
privacy-policy
```

Aliases accepted by service include `terms`, `terms-and-condition`, `privacy`, and `privacy-policy`.

### Public Get Content By Type

```http
GET /legal-content/public/:type
GET /legal-content/:type
```

Public. Returns published legal contents by type.

### Public Get Content By Id

```http
GET /legal-content/public/:type/:contentId
GET /legal-content/:type/:contentId
```

Public. Returns one published legal content item.

### Admin Get All Content

```http
GET /admin/legal-content
```

Protected admin.

### Admin Create Content

```http
POST /admin/legal-content/:type
```

Body:

```json
{
  "title": "Terms and Conditions",
  "contentHtml": "<p>Terms...</p>",
  "contentDelta": null,
  "plainText": "Terms...",
  "isPublished": true
}
```

Required: `title`, `contentHtml`.

### Admin Get Content By Type

```http
GET /admin/legal-content/:type
```

### Admin Get Content By Id

```http
GET /admin/legal-content/:type/:contentId
```

### Admin Update Content

```http
PATCH /admin/legal-content/:type/:contentId
```

Body:

```json
{
  "title": "Updated title",
  "contentHtml": "<p>Updated...</p>",
  "contentDelta": null,
  "plainText": "Updated...",
  "isPublished": true
}
```

All fields are optional, but empty `title` or empty `contentHtml` is rejected if sent.

### Admin Delete Content

```http
DELETE /admin/legal-content/:type/:contentId
```

## Socket.IO Events

Socket auth:

```js
import { io } from "socket.io-client";

const socket = io("https://your-api-domain.com", {
  transports: ["websocket"],
  auth: {
    token: accessToken
  }
});
```

On successful connection, backend joins the user to `user:<userId>` room and emits:

```js
socket.on("socket:ready", ({ userId, role, room }) => {});
```

### Ride Request Socket Events

Client emits:

```js
socket.emit("ride-request:sync");
```

Server emits:

```js
socket.on("ride-request:queue", ({ requests, radiusMiles, radiusKm, triggeredBy }) => {});
socket.on("ride-request:error", ({ message }) => {});
socket.on("ride-request:new", ({ request }) => {});
socket.on("ride-request:updated", ({ request, quote, nearbyDriverCount }) => {});
socket.on("ride-request:removed", ({ requestId, reason }) => {});
socket.on("ride-request:created", ({ request, nearbyDriverCount }) => {});
socket.on("ride-request:matched", (payload) => {});
socket.on("ride-request:accepted", ({ requestId, trip }) => {});
```

### Trip Socket Events

Server emits:

```js
socket.on("trip:driver-location", ({ tripId, driverId, status, progress }) => {});
socket.on("trip:destination-changed", ({ tripId, trip, newFare }) => {});
socket.on("trip:cancelled", (payload) => {});
socket.on("trip:completed", ({ trip, paymentSummary, autoCharge }) => {});
```

### Trip Chat Socket Events

Client emits:

```js
socket.emit("chat:join", { tripId });
socket.emit("chat:leave", { tripId });
socket.emit("chat:send", {
  tripId,
  text: "I am at the gate",
  tempId: "local-temp-id"
});
socket.emit("chat:seen", { tripId });
```

Server emits:

```js
socket.on("chat:joined", ({ tripId, room }) => {});
socket.on("chat:sent", ({ tempId, message }) => {});
socket.on("chat:new", (message) => {});
socket.on("chat:seen:update", (payload) => {});
socket.on("chat:error", ({ message, tempId }) => {});
```

## Important Data Conventions

### Coordinates

Requests usually accept:

```json
{
  "lat": 23.7806,
  "lng": 90.4193
}
```

Database/API responses use GeoJSON:

```json
{
  "point": {
    "type": "Point",
    "coordinates": [90.4193, 23.7806]
  }
}
```

Coordinates are `[lng, lat]` in GeoJSON.

### Trip Statuses

```txt
accepted
driver_arrived
otp_verified
started
completed
cancelled
```

### Ride Request Statuses

```txt
searching
matched
expired
cancelled
```

### Payment Statuses

```txt
unpaid
authorized
paid
failed
refunded
partial
```

### User Roles

```txt
rider
driver
admin
```

