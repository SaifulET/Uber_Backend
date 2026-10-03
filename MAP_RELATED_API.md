# Map Related API Documentation

This file covers the backend APIs and Socket.IO events that a mobile app or frontend map screen needs for nearby drivers, ride requests, driver live location, route/fare changes, and trip progress.

Base URL examples in this document use:

```txt
https://your-api-domain.com
```

All HTTP APIs below require:

```http
Authorization: Bearer <access_token>
Content-Type: application/json
```

GeoJSON coordinates are stored by the backend as `[lng, lat]`, but request bodies usually accept `lat` and `lng` as separate fields.

## Common Location Payload

Use this shape anywhere the API asks for `pickup` or `dropoff`.

```json
{
  "address": "Gulshan 1, Dhaka",
  "lat": 23.7806,
  "lng": 90.4193,
  "label": "Home"
}
```

`label` is optional and mainly used when saving recent places. `address`, `lat`, and `lng` are required.

Backend location response shape:

```json
{
  "address": "Gulshan 1, Dhaka",
  "point": {
    "type": "Point",
    "coordinates": [90.4193, 23.7806]
  }
}
```

## Rider Map APIs

### 1. Get Rider Home

```http
GET /riderGetRide/home
```

Description: Fetches rider profile summary, saved recent places, payment readiness, active ride request, and active trip. Useful when loading the rider map home screen.

Payload: none.

Response:

```json
{
  "success": true,
  "message": "Rider home fetched successfully",
  "data": {
    "profile": {
      "_id": "userId",
      "name": "Rider Name",
      "profileImage": null
    },
    "paymentMethod": {
      "ready": true,
      "customerId": "cus_xxx",
      "defaultPaymentMethodId": "pm_xxx"
    },
    "recentPlaces": [
      {
        "label": "Home",
        "address": "Gulshan 1, Dhaka",
        "location": {
          "type": "Point",
          "coordinates": [90.4193, 23.7806]
        }
      }
    ],
    "activeRequest": null,
    "activeTrip": null
  }
}
```

### 2. Get Recent Places

```http
GET /riderGetRide/recent-places
```

Description: Returns up to 10 recent pickup/dropoff places saved from previous ride requests.

Payload: none.

Response:

```json
{
  "success": true,
  "message": "Recent places fetched successfully",
  "data": {
    "places": [
      {
        "label": "Office",
        "address": "Banani, Dhaka",
        "location": {
          "type": "Point",
          "coordinates": [90.4001, 23.7937]
        }
      }
    ]
  }
}
```

### 3. Get Ride Options / Fare Estimate

```http
POST /riderGetRide/ride-options
```

Description: Returns available vehicle options and estimated fare using distance and duration calculated by the frontend or map SDK.

Payload:

```json
{
  "estimatedMiles": 6.2,
  "estimatedMinutes": 24
}
```

Response:

```json
{
  "success": true,
  "message": "Ride options fetched successfully",
  "data": {
    "currency": "USD",
    "options": [
      {
        "vehicleType": "car",
        "tier": "regular",
        "size": "normal",
        "quote": {
          "currency": "USD",
          "baseFare": 2.5,
          "estimatedMiles": 6.2,
          "estimatedMinutes": 24,
          "estimatedFare": 18.7,
          "pricePerMile": 2.5,
          "pricePerMinute": 0.25,
          "driverSharePercent": 80
        }
      }
    ]
  }
}
```

Vehicle options include `car`, `suv`, and `van`; tiers include `regular` and `premium`.

### 4. Get Nearby Online Drivers

```http
POST /riderGetRide/nearby-drivers
```

Description: Finds online, available, dispatch-eligible drivers near a map point. The search radius is 10 km.

Payload:

```json
{
  "lat": 23.7806,
  "lng": 90.4193
}
```

Response:

```json
{
  "success": true,
  "message": "Nearby online drivers fetched successfully",
  "data": {
    "drivers": [
      {
        "driverId": "driverUserId",
        "name": "Driver Name",
        "profileImage": null,
        "ratingAvg": 4.8,
        "ratingCount": 120,
        "location": {
          "point": {
            "type": "Point",
            "coordinates": [90.418, 23.781]
          },
          "updatedAt": "2026-10-03T08:10:00.000Z"
        },
        "vehicle": {
          "_id": "vehicleId",
          "brand": "Toyota",
          "model": "Prius",
          "type": "car",
          "tier": "regular",
          "size": "normal",
          "seats": 4,
          "licensePlate": "DHA-1234"
        }
      }
    ],
    "radiusKm": 10
  }
}
```

### 5. Create Ride Request

```http
POST /riderGetRide/ride-request
```

Description: Creates a ride request from pickup/dropoff points. The rider must have a saved card. The backend saves recent places, emits realtime events to the rider, and emits new ride request events to nearby eligible drivers.

Payload:

```json
{
  "pickup": {
    "address": "Gulshan 1, Dhaka",
    "lat": 23.7806,
    "lng": 90.4193,
    "label": "Pickup"
  },
  "dropoff": {
    "address": "Bashundhara City, Dhaka",
    "lat": 23.7516,
    "lng": 90.3906,
    "label": "Mall"
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
  "estimatedMiles": 6.2,
  "estimatedMinutes": 24
}
```

Response:

```json
{
  "success": true,
  "message": "Ride request created successfully",
  "data": {
    "rideRequest": {
      "_id": "requestId",
      "riderId": {
        "_id": "riderUserId",
        "name": "Rider Name",
        "profileImage": null,
        "ratingAvg": 4.9,
        "ratingCount": 32
      },
      "pickup": {
        "address": "Gulshan 1, Dhaka",
        "point": {
          "type": "Point",
          "coordinates": [90.4193, 23.7806]
        }
      },
      "dropoff": {
        "address": "Bashundhara City, Dhaka",
        "point": {
          "type": "Point",
          "coordinates": [90.3906, 23.7516]
        }
      },
      "schedule": {
        "kind": "now",
        "pickupAt": null
      },
      "preference": {
        "vehicleType": "car",
        "tier": "regular",
        "seats": 4,
        "size": "normal"
      },
      "status": "searching",
      "quote": {
        "currency": "USD",
        "estimatedMiles": 6.2,
        "estimatedMinutes": 24,
        "baseFare": 2.5,
        "estimatedFare": 18.7,
        "driverSharePercent": 80
      },
      "expiresAt": "2026-10-03T08:20:00.000Z"
    },
    "nearbyDrivers": [],
    "recentPlacesCount": 8
  }
}
```

Possible errors:

```json
{ "success": false, "message": "Add a card before requesting a trip", "error": null }
```

```json
{ "success": false, "message": "You already have an active request or trip", "error": null }
```

### 6. Get Active Rider Request / Trip

```http
GET /riderGetRide/active
```

Description: Returns the rider's currently searching ride request and/or active trip. Use this to restore map state after app restart.

Payload: none.

Response:

```json
{
  "success": true,
  "message": "Active ride data fetched successfully",
  "data": {
    "activeRequest": null,
    "activeTrip": {
      "_id": "tripId",
      "status": "accepted",
      "pickup": {
        "address": "Gulshan 1, Dhaka",
        "point": { "type": "Point", "coordinates": [90.4193, 23.7806] }
      },
      "dropoff": {
        "address": "Bashundhara City, Dhaka",
        "point": { "type": "Point", "coordinates": [90.3906, 23.7516] }
      }
    }
  }
}
```

### 7. Change Ride Request Pickup / Dropoff

```http
PATCH /riderGetRide/ride-request/:requestId/change-destination
```

Description: Updates pickup and/or dropoff while the request is still active. If pickup changes, nearby driver eligibility is recalculated and drivers receive `ride-request:new`, `ride-request:removed`, or `ride-request:updated`.

Payload:

```json
{
  "pickup": {
    "address": "Banani 11, Dhaka",
    "lat": 23.7937,
    "lng": 90.4001
  },
  "dropoff": {
    "address": "Dhanmondi 27, Dhaka",
    "lat": 23.7461,
    "lng": 90.3742
  },
  "estimatedMiles": 7.1,
  "estimatedMinutes": 28
}
```

You may send only `pickup` or only `dropoff`.

Response:

```json
{
  "success": true,
  "message": "Ride request locations changed successfully",
  "data": {
    "message": "Ride request locations changed successfully",
    "rideRequest": {
      "_id": "requestId",
      "status": "searching"
    },
    "newQuote": {
      "currency": "USD",
      "estimatedMiles": 7.1,
      "estimatedMinutes": 28,
      "estimatedFare": 21.25
    },
    "nearbyDrivers": []
  }
}
```

### 8. Check Fare Before Changing Ride Request

```http
POST /riderGetRide/ride-request/:requestId/check-fare
```

Description: Calculates a new fare for a changed pickup/dropoff without saving it.

Payload:

```json
{
  "dropoff": {
    "address": "Airport, Dhaka",
    "lat": 23.8433,
    "lng": 90.3978
  },
  "estimatedMiles": 9.5,
  "estimatedMinutes": 35
}
```

Response:

```json
{
  "success": true,
  "message": "Ride request fare checked successfully",
  "data": {
    "requestId": "requestId",
    "currentQuote": {
      "currency": "USD",
      "estimatedMiles": 6.2,
      "estimatedMinutes": 24,
      "estimatedFare": 18.7
    },
    "checkedQuote": {
      "currency": "USD",
      "estimatedMiles": 9.5,
      "estimatedMinutes": 35,
      "estimatedFare": 28.5
    },
    "pickup": {
      "address": "Gulshan 1, Dhaka",
      "point": { "type": "Point", "coordinates": [90.4193, 23.7806] }
    },
    "dropoff": {
      "address": "Airport, Dhaka",
      "point": { "type": "Point", "coordinates": [90.3978, 23.8433] }
    },
    "preference": {
      "vehicleType": "car",
      "tier": "regular",
      "seats": 4,
      "size": "normal"
    }
  }
}
```

### 9. Cancel Ride Request

```http
PATCH /riderGetRide/ride-request/:requestId/cancel
```

Description: Cancels an active ride request. Nearby drivers receive `ride-request:removed`.

Payload: none.

Response:

```json
{
  "success": true,
  "message": "Ride request cancelled successfully",
  "data": {
    "message": "Ride request cancelled successfully",
    "rideRequest": {
      "_id": "requestId",
      "status": "cancelled"
    }
  }
}
```

### 10. Check Fare During Active Trip

```http
POST /riderGetRide/trip/:tripId/check-fare
```

Description: Calculates fare if pickup/dropoff changes during an active trip. Pickup cannot be changed after the trip has started when using the actual change endpoint.

Payload:

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

Response:

```json
{
  "success": true,
  "message": "Fare checked successfully",
  "data": {
    "tripId": "tripId",
    "currentFare": {
      "currency": "USD",
      "estimatedFare": 18.7,
      "finalFare": 18.7
    },
    "checkedFare": {
      "currency": "USD",
      "estimatedMiles": 12,
      "estimatedMinutes": 42,
      "estimatedFare": 35.4
    },
    "pickup": {
      "address": "Gulshan 1, Dhaka",
      "point": { "type": "Point", "coordinates": [90.4193, 23.7806] }
    },
    "dropoff": {
      "address": "Airport, Dhaka",
      "point": { "type": "Point", "coordinates": [90.3978, 23.8433] }
    },
    "rideOption": {
      "vehicleType": "car",
      "tier": "regular",
      "seats": 4,
      "size": "normal"
    }
  }
}
```

### 11. Change Active Trip Pickup / Dropoff

```http
PATCH /riderGetRide/trip/:tripId/change-destination
```

Description: Saves changed pickup and/or dropoff for an active trip and emits `trip:destination-changed` to both rider and driver.

Payload:

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

Response:

```json
{
  "success": true,
  "message": "Trip locations changed successfully",
  "data": {
    "message": "Trip locations changed successfully",
    "trip": {
      "_id": "tripId",
      "status": "accepted",
      "dropoff": {
        "address": "Airport, Dhaka",
        "point": { "type": "Point", "coordinates": [90.3978, 23.8433] }
      }
    },
    "newFare": {
      "currency": "USD",
      "estimatedMiles": 12,
      "estimatedMinutes": 42,
      "estimatedFare": 35.4
    }
  }
}
```

### 12. Get Trip Details

```http
GET /riderGetRide/trip/:tripId/details
```

Description: Returns trip details for the rider, including pickup/dropoff, driver, vehicle, fare, status, and review state.

Payload: none.

Response:

```json
{
  "success": true,
  "message": "Trip details fetched successfully",
  "data": {
    "trip": {
      "_id": "tripId",
      "status": "accepted",
      "pickup": {
        "address": "Gulshan 1, Dhaka",
        "point": { "type": "Point", "coordinates": [90.4193, 23.7806] }
      },
      "dropoff": {
        "address": "Airport, Dhaka",
        "point": { "type": "Point", "coordinates": [90.3978, 23.8433] }
      }
    },
    "driver": {
      "_id": "driverUserId",
      "name": "Driver Name",
      "profileImage": null,
      "ratingAvg": 4.8,
      "ratingCount": 120
    },
    "vehicle": {
      "_id": "vehicleId",
      "brand": "Toyota",
      "model": "Prius",
      "type": "car",
      "size": "normal",
      "licensePlate": "DHA-1234"
    }
  }
}
```

## Driver Map APIs

### 1. Driver Go Online

```http
PATCH /driverHome/go-online
```

Description: Marks driver online and optionally updates current location. Driver must be active, documents must be complete, and an active vehicle is required.

Payload:

```json
{
  "lat": 23.7806,
  "lng": 90.4193
}
```

Response:

```json
{
  "success": true,
  "message": "Driver is now online",
  "data": {
    "message": "Driver is now online",
    "isOnline": true,
    "isBusy": false,
    "location": {
      "point": {
        "type": "Point",
        "coordinates": [90.4193, 23.7806]
      },
      "updatedAt": "2026-10-03T08:10:00.000Z"
    }
  }
}
```

### 2. Driver Go Offline

```http
PATCH /driverHome/go-offline
```

Description: Marks driver offline. Driver cannot go offline during an active trip.

Payload: none.

Response:

```json
{
  "success": true,
  "message": "Driver is now offline",
  "data": {
    "message": "Driver is now offline",
    "isOnline": false,
    "isBusy": false
  }
}
```

### 3. Update Driver Location

```http
PATCH /driverHome/location
```

Description: Updates driver live location. If the driver has an active trip, the rider receives `trip:driver-location` with ETA/distance progress.

Payload:

```json
{
  "lat": 23.781,
  "lng": 90.418
}
```

Response:

```json
{
  "success": true,
  "message": "Driver location updated",
  "data": {
    "message": "Driver location updated",
    "location": {
      "point": {
        "type": "Point",
        "coordinates": [90.418, 23.781]
      },
      "updatedAt": "2026-10-03T08:11:00.000Z"
    }
  }
}
```

Suggested driver app behavior: call this endpoint when the driver's GPS changes meaningfully, for example every 5-10 seconds during a trip or every 20-30 seconds while idle online.

### 4. Get Next Ride Request

```http
GET /driverHome/ride-requests/next
```

Description: Returns the oldest nearby searching ride request within 10 km for the online, available driver.

Payload: none.

Response:

```json
{
  "success": true,
  "message": "Next ride request fetched successfully",
  "data": {
    "request": {
      "_id": "requestId",
      "riderId": {
        "_id": "riderUserId",
        "name": "Rider Name",
        "profileImage": null,
        "ratingAvg": 4.9,
        "ratingCount": 32
      },
      "pickup": {
        "address": "Gulshan 1, Dhaka",
        "point": { "type": "Point", "coordinates": [90.4193, 23.7806] }
      },
      "dropoff": {
        "address": "Airport, Dhaka",
        "point": { "type": "Point", "coordinates": [90.3978, 23.8433] }
      },
      "status": "searching",
      "quote": {
        "currency": "USD",
        "estimatedMiles": 12,
        "estimatedMinutes": 42,
        "estimatedFare": 35.4
      }
    }
  }
}
```

If unavailable:

```json
{
  "success": true,
  "data": {
    "request": null
  }
}
```

### 5. Get Nearby Ride Requests

```http
GET /driverHome/ride-requests/nearby
```

Description: Returns all nearby searching ride requests within 10 km for the online, available driver.

Payload: none.

Response:

```json
{
  "success": true,
  "message": "Nearby ride requests fetched successfully",
  "data": {
    "requests": [
      {
        "_id": "requestId",
        "pickup": {
          "address": "Gulshan 1, Dhaka",
          "point": { "type": "Point", "coordinates": [90.4193, 23.7806] }
        },
        "dropoff": {
          "address": "Airport, Dhaka",
          "point": { "type": "Point", "coordinates": [90.3978, 23.8433] }
        },
        "quote": {
          "currency": "USD",
          "estimatedFare": 35.4
        }
      }
    ],
    "radiusMiles": 6.2137,
    "radiusKm": 10
  }
}
```

### 6. Accept Ride Request

```http
PATCH /driverHome/ride-requests/:requestId/accept
```

Description: Driver accepts a searching ride request. Backend creates a trip, marks driver busy, sends `ride-request:matched` to rider, sends `ride-request:accepted` to driver, and removes the request from other nearby drivers.

Payload: none.

Response:

```json
{
  "success": true,
  "message": "Ride request accepted successfully",
  "data": {
    "message": "Ride request accepted successfully",
    "trip": {
      "_id": "tripId",
      "requestId": "requestId",
      "riderId": "riderUserId",
      "driverId": "driverUserId",
      "vehicleId": "vehicleId",
      "pickup": {
        "address": "Gulshan 1, Dhaka",
        "point": { "type": "Point", "coordinates": [90.4193, 23.7806] }
      },
      "dropoff": {
        "address": "Airport, Dhaka",
        "point": { "type": "Point", "coordinates": [90.3978, 23.8433] }
      },
      "status": "accepted",
      "otp": {
        "hash": "123456",
        "expiresAt": "2026-10-03T08:25:00.000Z",
        "verifiedAt": null
      },
      "pricing": {
        "currency": "USD",
        "estimatedMiles": 12,
        "estimatedMinutes": 42,
        "estimatedFare": 35.4,
        "finalFare": 35.4
      }
    }
  }
}
```

### 7. Get Driver Active Trip

```http
GET /driverHome/trip/active
```

Description: Returns the driver's active trip for map restoration.

Payload: none.

Response:

```json
{
  "success": true,
  "message": "Active trip fetched successfully",
  "data": {
    "trip": {
      "_id": "tripId",
      "status": "accepted",
      "pickup": {
        "address": "Gulshan 1, Dhaka",
        "point": { "type": "Point", "coordinates": [90.4193, 23.7806] }
      },
      "dropoff": {
        "address": "Airport, Dhaka",
        "point": { "type": "Point", "coordinates": [90.3978, 23.8433] }
      },
      "riderId": {
        "_id": "riderUserId",
        "name": "Rider Name",
        "profileImage": null,
        "ratingAvg": 4.9,
        "ratingCount": 32
      }
    }
  }
}
```

### 8. Driver Arrived At Pickup

```http
PATCH /driverHome/trip/:tripId/arrived-pickup
```

Description: Moves trip from `accepted` to `driver_arrived`.

Payload: none.

Response:

```json
{
  "success": true,
  "message": "Driver arrived at pickup successfully",
  "data": {
    "message": "Driver arrived at pickup successfully",
    "trip": {
      "_id": "tripId",
      "status": "driver_arrived"
    }
  }
}
```

### 9. Verify OTP And Start Trip

```http
PATCH /driverHome/trip/:tripId/verify-otp
```

Description: Verifies rider OTP and starts the trip.

Payload:

```json
{
  "otp": "123456"
}
```

Response:

```json
{
  "success": true,
  "message": "OTP verified and trip started successfully",
  "data": {
    "message": "OTP verified and trip started successfully",
    "trip": {
      "_id": "tripId",
      "status": "started"
    }
  }
}
```

### 10. Start Trip

```http
PATCH /driverHome/trip/:tripId/start
```

Description: Starts a trip that is already arrived/OTP-ready. Normally `verify-otp` is the safer endpoint because it validates the rider OTP.

Payload: none.

Response:

```json
{
  "success": true,
  "message": "Trip started successfully",
  "data": {
    "message": "Trip started successfully",
    "trip": {
      "_id": "tripId",
      "status": "started"
    }
  }
}
```

### 11. Complete Trip

```http
PATCH /driverHome/trip/:tripId/complete
```

Description: Completes a started trip, calculates final fare from trip duration and distance, frees the driver, and emits `trip:completed` to rider and driver.

Payload:

```json
{
  "distanceMiles": 12.4
}
```

`distanceMiles` is optional. If omitted, backend uses the trip's estimated or saved distance.

Response:

```json
{
  "success": true,
  "message": "Trip completed and rider card charged successfully",
  "data": {
    "message": "Trip completed and rider card charged successfully",
    "trip": {
      "_id": "tripId",
      "status": "completed",
      "paymentStatus": "paid",
      "distanceMiles": 12.4,
      "durationMinutes": 38
    },
    "paymentSummary": {
      "totalFare": 34.8,
      "driverGets": 27.84,
      "platformGets": 6.96,
      "paymentStatus": "paid"
    },
    "autoCharge": {
      "attempted": true,
      "succeeded": true
    }
  }
}
```

### 12. Cancel Trip By Driver

```http
PATCH /driverHome/trip/:tripId/cancel
```

Description: Cancels an active trip by driver, frees the driver, and emits `trip:cancelled` to rider and driver.

Payload:

```json
{
  "reason": "Vehicle issue"
}
```

Response:

```json
{
  "success": true,
  "message": "Trip cancelled successfully",
  "data": {
    "message": "Trip cancelled successfully",
    "trip": {
      "_id": "tripId",
      "status": "cancelled",
      "cancellation": {
        "canceledBy": "driver",
        "reason": "Vehicle issue",
        "feeCharged": 0,
        "rule": "DRIVER_CANCEL"
      }
    }
  }
}
```

## Socket.IO Integration

Socket server is initialized with the same HTTP server. The client must authenticate during connection.

```js
import { io } from "socket.io-client";

const socket = io("https://your-api-domain.com", {
  transports: ["websocket"],
  auth: {
    token: accessToken
  }
});

socket.on("connect", () => {
  console.log("connected", socket.id);
});

socket.on("connect_error", (error) => {
  console.log("socket auth/connect error", error.message);
});

socket.on("socket:ready", (payload) => {
  console.log("socket ready", payload);
  // { userId, role, room: "user:<userId>" }
});
```

You can also send the token as an authorization header:

```js
const socket = io("https://your-api-domain.com", {
  extraHeaders: {
    Authorization: `Bearer ${accessToken}`
  }
});
```

### Rider Socket Events To Listen For

#### `ride-request:created`

Emitted to the rider after creating a ride request.

```js
socket.on("ride-request:created", ({ request, nearbyDriverCount }) => {
  // show searching state on map
});
```

Payload:

```json
{
  "request": {
    "_id": "requestId",
    "status": "searching",
    "pickup": {
      "address": "Gulshan 1, Dhaka",
      "point": { "type": "Point", "coordinates": [90.4193, 23.7806] }
    },
    "dropoff": {
      "address": "Airport, Dhaka",
      "point": { "type": "Point", "coordinates": [90.3978, 23.8433] }
    }
  },
  "nearbyDriverCount": 3
}
```

#### `ride-request:matched`

Emitted to the rider when a driver accepts the request.

```js
socket.on("ride-request:matched", (payload) => {
  // draw driver marker, route driver to pickup, show OTP
});
```

Payload:

```json
{
  "requestId": "requestId",
  "matchedDriverId": "driverUserId",
  "trip": {
    "_id": "tripId",
    "status": "accepted"
  },
  "driver": {
    "_id": "driverUserId",
    "name": "Driver Name",
    "profileImage": null,
    "ratingAvg": 4.8,
    "ratingCount": 120,
    "tripsCount": 80,
    "phone": "+880..."
  },
  "vehicle": {
    "_id": "vehicleId",
    "brand": "Toyota",
    "model": "Prius",
    "type": "car",
    "size": "normal",
    "licensePlate": "DHA-1234"
  },
  "fare": {
    "currency": "USD",
    "estimatedFare": 35.4,
    "finalFare": 35.4
  },
  "otp": {
    "code": "123456",
    "expiresAt": "2026-10-03T08:25:00.000Z"
  },
  "pickupProgress": {
    "target": "pickup",
    "currentLocation": {
      "point": { "type": "Point", "coordinates": [90.418, 23.781] },
      "updatedAt": "2026-10-03T08:11:00.000Z"
    },
    "distanceMeters": 320,
    "distanceKm": 0.32,
    "etaMinutes": 2,
    "updatedAt": "2026-10-03T08:11:00.000Z"
  },
  "note": {
    "pickupInstruction": "Meet at the pickup location",
    "otpInstruction": "Share OTP with driver to start the trip",
    "cancellationRule": "If cancelled late, up to 60% of the fare may be charged"
  }
}
```

#### `trip:driver-location`

Emitted to the rider whenever the driver updates location during an active trip.

```js
socket.on("trip:driver-location", ({ tripId, driverId, status, progress }) => {
  const [lng, lat] = progress.currentLocation.point.coordinates;
  // move driver marker to { lat, lng }
  // if status === "started", target is dropoff; otherwise target is pickup
});
```

Payload:

```json
{
  "tripId": "tripId",
  "driverId": "driverUserId",
  "status": "accepted",
  "progress": {
    "target": "pickup",
    "currentLocation": {
      "point": {
        "type": "Point",
        "coordinates": [90.418, 23.781]
      },
      "updatedAt": "2026-10-03T08:11:00.000Z"
    },
    "distanceMeters": 320,
    "distanceKm": 0.32,
    "etaMinutes": 2,
    "updatedAt": "2026-10-03T08:11:00.000Z"
  }
}
```

#### Other Rider Events

```js
socket.on("ride-request:updated", ({ request, quote, nearbyDriverCount }) => {});
socket.on("ride-request:cancelled", ({ requestId }) => {});
socket.on("trip:destination-changed", ({ tripId, trip, newFare }) => {});
socket.on("trip:cancelled", ({ tripId, trip, cancellation }) => {});
socket.on("trip:completed", ({ trip, paymentSummary, autoCharge }) => {});
```

### Driver Socket Events To Listen For

#### `ride-request:queue`

Emitted when the driver connects, goes online, updates location, accepts/cancels/completes a trip, or manually syncs.

```js
socket.on("ride-request:queue", ({ requests, radiusMiles, radiusKm, triggeredBy }) => {
  // render request pins/list for the driver
});
```

Payload:

```json
{
  "requests": [
    {
      "_id": "requestId",
      "riderId": {
        "_id": "riderUserId",
        "name": "Rider Name",
        "profileImage": null,
        "ratingAvg": 4.9,
        "ratingCount": 32
      },
      "pickup": {
        "address": "Gulshan 1, Dhaka",
        "point": { "type": "Point", "coordinates": [90.4193, 23.7806] }
      },
      "dropoff": {
        "address": "Airport, Dhaka",
        "point": { "type": "Point", "coordinates": [90.3978, 23.8433] }
      },
      "status": "searching",
      "quote": {
        "currency": "USD",
        "estimatedMiles": 12,
        "estimatedMinutes": 42,
        "estimatedFare": 35.4
      }
    }
  ],
  "radiusMiles": 6.2137,
  "radiusKm": 10,
  "triggeredBy": "socket_connected"
}
```

Manual sync from driver:

```js
socket.emit("ride-request:sync");
```

Possible error:

```js
socket.on("ride-request:error", ({ message }) => {
  console.log(message);
});
```

#### `ride-request:new`

Emitted to eligible nearby drivers when a new rider request is created or pickup changes into the driver's area.

```js
socket.on("ride-request:new", ({ request }) => {
  // add request marker/card
});
```

#### `ride-request:updated`

Emitted when a request location/fare changes.

```js
socket.on("ride-request:updated", ({ request, quote }) => {
  // update marker/card/fare
});
```

#### `ride-request:removed`

Emitted when the request is cancelled, matched by another driver, or pickup moves out of range.

```js
socket.on("ride-request:removed", ({ requestId, reason }) => {
  // remove request marker/card
});
```

Reasons include `cancelled`, `matched`, and `pickup_changed`.

#### `ride-request:accepted`

Emitted to the accepting driver after accept succeeds.

```js
socket.on("ride-request:accepted", ({ requestId, trip }) => {
  // navigate to active trip map
});
```

#### Other Driver Events

```js
socket.on("trip:destination-changed", ({ tripId, trip, newFare }) => {});
socket.on("trip:cancelled", ({ tripId, trip, cancellation }) => {});
socket.on("trip:completed", ({ trip, paymentSummary, autoCharge }) => {});
```

## Trip Chat Socket Events

These are not map-specific, but they are useful on the active trip screen.

Join trip chat:

```js
socket.emit("chat:join", { tripId });
```

Leave trip chat:

```js
socket.emit("chat:leave", { tripId });
```

Send message:

```js
socket.emit("chat:send", {
  tripId,
  text: "I am at the gate",
  tempId: "local-temp-id-1"
});
```

Mark seen:

```js
socket.emit("chat:seen", { tripId });
```

Listen:

```js
socket.on("chat:joined", ({ tripId, room }) => {});
socket.on("chat:sent", ({ tempId, message }) => {});
socket.on("chat:new", (message) => {});
socket.on("chat:seen:update", (payload) => {});
socket.on("chat:error", ({ message, tempId }) => {});
```

## Frontend Helper Functions

### Convert Backend GeoJSON To Map Coordinates

```js
export function pointToLatLng(point) {
  if (!point?.coordinates || point.coordinates.length !== 2) {
    return null;
  }

  const [lng, lat] = point.coordinates;
  return { lat, lng };
}
```

### Rider Flow Example

```js
async function loadRiderMap(api) {
  const home = await api.get("/riderGetRide/home");

  if (home.data.activeTrip) {
    return { mode: "active_trip", trip: home.data.activeTrip };
  }

  if (home.data.activeRequest) {
    return { mode: "searching", request: home.data.activeRequest };
  }

  return { mode: "idle", recentPlaces: home.data.recentPlaces };
}

async function requestRide(api, routeInfo, pickup, dropoff, selectedOption) {
  return api.post("/riderGetRide/ride-request", {
    pickup,
    dropoff,
    schedule: { kind: "now", pickupAt: null },
    preference: {
      vehicleType: selectedOption.vehicleType,
      tier: selectedOption.tier,
      seats: selectedOption.seats || 4
    },
    estimatedMiles: routeInfo.distanceMiles,
    estimatedMinutes: routeInfo.durationMinutes
  });
}
```

### Driver Flow Example

```js
async function goOnline(api, currentPosition) {
  return api.patch("/driverHome/go-online", {
    lat: currentPosition.coords.latitude,
    lng: currentPosition.coords.longitude
  });
}

async function updateDriverLocation(api, currentPosition) {
  return api.patch("/driverHome/location", {
    lat: currentPosition.coords.latitude,
    lng: currentPosition.coords.longitude
  });
}

async function acceptRequest(api, requestId) {
  return api.patch(`/driverHome/ride-requests/${requestId}/accept`);
}
```

### React Native Location Watch Example

```js
let watchId = null;

export function startDriverLocationWatch(api, Geolocation) {
  watchId = Geolocation.watchPosition(
    async (position) => {
      await api.patch("/driverHome/location", {
        lat: position.coords.latitude,
        lng: position.coords.longitude
      });
    },
    (error) => console.log("location error", error),
    {
      enableHighAccuracy: true,
      distanceFilter: 20,
      interval: 8000,
      fastestInterval: 5000
    }
  );
}

export function stopDriverLocationWatch(Geolocation) {
  if (watchId !== null) {
    Geolocation.clearWatch(watchId);
    watchId = null;
  }
}
```

## Map SDK Responsibilities

The backend does not call Google Maps/Mapbox directions directly in these APIs. The app/frontend should calculate:

- pickup/dropoff address from places/autocomplete/geocoding
- route polyline
- route distance in miles
- route duration in minutes

Then send `estimatedMiles` and `estimatedMinutes` to fare-related APIs.

Recommended frontend sequence for rider:

1. Use map autocomplete/geocoding to select pickup and dropoff.
2. Use directions API to calculate route, distance, and duration.
3. Call `POST /riderGetRide/ride-options`.
4. Show available options and nearby drivers from `POST /riderGetRide/nearby-drivers`.
5. Call `POST /riderGetRide/ride-request`.
6. Listen for `ride-request:matched` and then `trip:driver-location`.

Recommended frontend sequence for driver:

1. Connect socket with auth token.
2. Call `PATCH /driverHome/go-online` with current GPS.
3. Listen for `ride-request:queue` and `ride-request:new`.
4. Accept using `PATCH /driverHome/ride-requests/:requestId/accept`.
5. Keep sending `PATCH /driverHome/location`.
6. Mark arrival, verify OTP, complete trip.

