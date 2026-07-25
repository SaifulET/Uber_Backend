# Uber Backend Code Documentation

## Short Summary

This is the Node.js/Express backend for a ride-sharing platform. It provides rider APIs, driver APIs, admin APIs, Stripe payment flows, AWS S3 image uploads, MongoDB persistence through Mongoose, and Socket.IO realtime events for ride requests and trip chat.

The backend is organized by business modules: auth, rider ride booking, driver home/trip workflow, driver onboarding, admin management, legal content, support tickets/reports, payments, and chat.

## Technology Stack

- Runtime: Node.js with ES modules
- Web framework: Express
- Database: MongoDB through Mongoose
- Realtime: Socket.IO
- Auth: JWT access and refresh tokens
- File upload: Multer + AWS S3
- Payments: Stripe customers, setup/payment intents, and Stripe Connect onboarding
- Email: Nodemailer SMTP
- Deployment: Dockerfile using `node:22-alpine`


## Main API Areas

### Public and User Auth

Mounted at `/auth`.

Key endpoints:

- `POST /auth/register`
- `POST /auth/login`
- `POST /auth/google`
- `PATCH /auth/editRole`
- `POST /auth/send-verification`
- `POST /auth/verify-email`
- `POST /auth/forgot-password`
- `POST /auth/verify-reset-otp`
- `POST /auth/reset-password`
- `POST /auth/refresh`
- `POST /auth/logout`
- `GET /auth/me`
- `PATCH /auth/change-password`
- `DELETE /auth/delete-account`
- `PATCH /auth/image-save`
- `PATCH /auth/profileInfo-update`

Auth supports email/password, Google login, OTP email verification, password reset, refresh tokens, profile updates, and soft account deletion.

### Rider Ride Flow

Mounted at `/riderGetRide`.

This module supports rider home data, recent places, ride options, nearby driver lookup, payment method setup, ride request creation, active ride state, trip details, destination changes, cancellation, payment, ratings, and support tickets.

Important endpoints:

- `GET /riderGetRide/home`
- `GET /riderGetRide/recent-places`
- `POST /riderGetRide/ride-options`
- `POST /riderGetRide/nearby-drivers`
- `GET /riderGetRide/payment-method`
- `POST /riderGetRide/payment-method/setup-intent`
- `POST /riderGetRide/payment-method/save`
- `POST /riderGetRide/ride-request`
- `GET /riderGetRide/active`
- `GET /riderGetRide/trips`
- `PATCH /riderGetRide/ride-request/:requestId/change-destination`
- `POST /riderGetRide/ride-request/:requestId/check-fare`
- `PATCH /riderGetRide/ride-request/:requestId/cancel`
- `PATCH /riderGetRide/trip/:tripId/cancel`
- `PATCH /riderGetRide/trip/:tripId/change-destination`
- `GET /riderGetRide/trip/:tripId/details`
- `GET /riderGetRide/trip/:tripId/payment-summary`
- `POST /riderGetRide/trip/:tripId/payment-intent`
- `POST /riderGetRide/trip/:tripId/payment-verify`
- `POST /riderGetRide/trip/:tripId/rating`
- `POST /riderGetRide/support-ticket`

### Driver Home and Trip Flow

Mounted at `/driverHome`.

This module manages driver online/offline state, live location, nearby ride request queue, accepting rides, trip status progression, OTP verification, completion, cancellation, earnings, reviews, and support tickets.

Important endpoints:

- `GET /driverHome/home`
- `PATCH /driverHome/go-online`
- `PATCH /driverHome/go-offline`
- `PATCH /driverHome/location`
- `GET /driverHome/ride-requests/next`
- `GET /driverHome/ride-requests/nearby`
- `PATCH /driverHome/ride-requests/:requestId/accept`
- `GET /driverHome/trip/active`
- `GET /driverHome/trips`
- `GET /driverHome/earnings-today`
- `GET /driverHome/earnings-summary`
- `PATCH /driverHome/trip/:tripId/arrived-pickup`
- `PATCH /driverHome/trip/:tripId/verify-otp`
- `PATCH /driverHome/trip/:tripId/start`
- `PATCH /driverHome/trip/:tripId/complete`
- `PATCH /driverHome/trip/:tripId/cancel`
- `GET /driverHome/trip/:tripId/summary`
- `POST /driverHome/support-ticket`

### Driver Onboarding

Mounted at `/driverOnboarding` and `/driverOnboardingRead`.

Driver onboarding stores vehicle details, profile photo, license front/back, vehicle registration, vehicle insurance, Stripe onboarding information, and review status.

Write endpoints include:

- `GET /driverOnboarding/status`
- `PATCH /driverOnboarding/status`
- `POST /driverOnboarding/vehicle`
- `POST /driverOnboarding/profile-photo`
- `POST /driverOnboarding/license-front`
- `POST /driverOnboarding/license-back`
- `POST /driverOnboarding/vehicle-registration`
- `POST /driverOnboarding/vehicle-insurance`
- `POST /driverOnboarding/stripe/onboarding-link`
- `POST /driverOnboarding/connect-stripe`
- `GET /driverOnboarding/review`

Read endpoints include:

- `GET /driverOnboardingRead/summary`
- `GET /driverOnboardingRead/profile-image`
- `GET /driverOnboardingRead/license-photos`
- `GET /driverOnboardingRead/vehicle-registration`
- `GET /driverOnboardingRead/vehicle-insurance`
- `GET /driverOnboardingRead/stripe`
- `GET /driverOnboardingRead/stripe/approval`
- `GET /driverOnboardingRead/vehicle-info`

### Admin APIs

Admin APIs are protected by JWT auth and require an admin user.

Mounted areas:

- `/admin/auth`: sign in, password reset, token refresh, logout, name/password change.
- `/admin/config`: fare configuration and payment share percentages.
- `/admin/dashboard`: overview and analytics metrics.
- `/admin/drivers`: driver listing, detail, document review, history, reports, status updates, delete/restore/hard delete.
- `/admin/riders`: rider listing, payments, detail, history, reports, status updates, delete/restore/hard delete.
- `/admin/customer-support`: support tickets and reports.
- `/admin/legal-content`: terms and privacy content management.

Public legal content is also mounted at `/legal-content`.

### Chat APIs

Mounted at `/chat`.

Endpoints:

- `GET /chat/trips/:tripId/chat/header`
- `GET /chat/trips/:tripId/chat/messages`
- `POST /chat/trips/:tripId/chat/messages`
- `PATCH /chat/trips/:tripId/chat/seen`
- `GET /chat/trips/:tripId/chat/unread-count`

## Realtime Behavior

Socket.IO is initialized in `src/messages/socket.js`.

Authentication:

- Socket clients must pass a JWT token through `socket.handshake.auth.token` or `Authorization: Bearer <token>`.
- Each connected user joins a room named `user:<userId>`.

Ride request events:

- Server emits `ride-request:queue` to eligible drivers.
- Driver clients can emit `ride-request:sync`.
- Queue eligibility requires active driver status, verified documents, no required actions, online state, not busy, and location availability.

Chat events:

- Client emits `chat:join`, `chat:leave`, `chat:send`, and `chat:seen`.
- Server emits `chat:joined`, `chat:sent`, `chat:error`, plus realtime message/seen payloads.

## Ride Matching

Ride matching uses geospatial MongoDB queries.

Important rules:

- Nearby drivers are searched within 10 kilometers.
- Driver profile must be active, online, not busy, document-verified, and linked to an active vehicle.
- Vehicle filters can include type, tier, and seats.
- Ride requests are queued by pickup proximity and expiration time.

## Data Model Overview

Main Mongoose models:

- `User`: shared user record for riders, drivers, and admins.
- `RiderProfile`: Stripe customer/payment method and saved places.
- `DriverProfile`: driver status, online/busy state, geolocation, document status, Stripe Connect, vehicle, earnings, and trip count.
- `Vehicle`: driver vehicle information, type/tier/size/seats, approval and review status.
- `DriverDocument`: uploaded driver onboarding documents.
- `RideRequest`: active rider request with pickup/dropoff, preference, quote, matched driver, status, and TTL expiry.
- `Trip`: accepted ride lifecycle, rider/driver/vehicle, OTP, status history, pricing, payment status, cancellation details.
- `Payment`: Stripe payment record and fare split between driver and platform.
- `Rating`: trip-based user ratings.
- `Message`: trip chat messages.
- `SupportTicket`: rider/driver support entries.
- `Report`: user reports against other users.
- `FareConfig`: active pricing configuration.
- `LegalContent`: terms and privacy content.
- `AuthToken`: OTP and reset token storage with TTL expiry.
- `DriverOnlineSession`: online session tracking for drivers.

## Environment Variables

Observed variables used by the backend:

- `PORT`
- `MONGO_URI`
- `DNS_SERVERS`
- `FRONTEND_URL`
- `API_URL`
- `ACCESS_TOKEN_SECRET`
- `JWT_ACCESS_SECRET`
- `JWT_SECRET`
- `REFRESH_TOKEN_SECRET`
- `JWT_REFRESH_SECRET`
- `ACCESS_EXPIRES_IN`
- `REFRESH_EXPIRES_IN`
- `OTP_TTL_MINUTES`
- `RESET_TOKEN_TTL_MINUTES`
- `GOOGLE_CLIENT_ID`
- `OTP_MIRROR_EMAIL`
- `SMTP_HOST`
- `SMTP_PORT`
- `SMTP_USER`
- `SMTP_PASS`
- `AWS_REGION`
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_BUCKET_NAME`
- `STRIPE_SECRET_KEY`
- `STRIPE_PUBLISHABLE_KEY`
- `STRIPE_CONNECT_COUNTRY`
- `MAASAD_STRIPE_SECRET_KEY_OVERRIDE`
- `MAASAD_STRIPE_PUBLISHABLE_KEY_OVERRIDE`

## Operational Notes

- The backend has no automated test scripts in `package.json`.
- The auth middleware logs the full Authorization header, which should be removed or reduced before production logging.
- Several protected routers call `requireAuth` both at router level and per route. This is redundant but usually harmless.
- Soft-delete fields exist on `User`; admin APIs also expose hard delete.
- `RideRequest` and `AuthToken` use TTL indexes.
- Stripe is abstracted to allow normal keys and special override keys for specific test emails.
- S3 upload middleware allows image uploads only; single upload uses the field name `image`.

## Run Commands

Install dependencies:

```bash
npm install
```

Development:

```bash
npm run dev
```

Production:

```bash
npm start
```

Docker:

```bash
docker build -t ma3-backend .
docker run -p 5001:5001 --env-file .env ma3-backend
```
