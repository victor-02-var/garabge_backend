# CivicSync -- Backend

Backend service for **CivicSync**, a municipal grievance reporting,
waste-management, and fleet-tracking platform.

The backend is responsible for complaint processing, AI-based
waste-image verification, EXIF GPS extraction, image storage, Supabase
database operations, vehicle assignment/tracking, authentication, and
administrative authorization.

> **Backend stack:** Node.js + Express.js\
> **Database:** Supabase PostgreSQL\
> **Image storage:** Cloudinary\
> **AI verification:** Google Gemini AI Vision\
> **Geospatial service:** OpenStreetMap Nominatim

## 1. Backend Responsibilities

The CivicSync backend connects the citizen/admin React applications with
external services and the application database.

Core responsibilities:

-   Receive citizen waste complaints.
-   Validate uploaded waste images using Gemini AI Vision.
-   Extract GPS coordinates from image EXIF metadata.
-   Support manual location fallback.
-   Upload complaint and resolution images to Cloudinary.
-   Store complaint data in Supabase PostgreSQL.
-   Authenticate users and authorize administrative operations.
-   Retrieve real vehicle records for assignment.
-   Track assigned municipal vehicles.
-   Update complaint status and resolution information.
-   Support route/geospatial operations used by the frontend.

The documented system workflow is:

``` text
Citizen Upload
      ↓
Express API
      ↓
Gemini AI Verification
      ↓
EXIF GPS Extraction
      ↓
Cloudinary Upload
      ↓
Supabase Complaint Record
      ↓
Admin Assignment
      ↓
Vehicle Tracking
      ↓
Resolution Proof
      ↓
Citizen Feedback
```

## 2. Technology Stack

  Component             Technology
  --------------------- -----------------------------------------
  Runtime               Node.js
  Web Framework         Express.js
  Database              Supabase PostgreSQL
  Object Storage        Cloudinary
  AI / Vision           Google Gemini AI Vision API
  File Upload           Multer buffers
  Authentication        JWT-based authentication
  Geolocation           EXIF metadata + OpenStreetMap Nominatim
  Frontend Consumers    React / Vite applications
  Development Sharing   Pinggy / Localtunnel

The project specification identifies Node.js and Express.js as the
backend technologies, Supabase PostgreSQL and Cloudinary for storage,
Gemini Vision for AI verification, and Nominatim for geocoding.
fileciteturn1file0L41-L49

## 3. Backend Architecture

``` text
                    React Clients
                         │
                         │ REST API
                         ↓
                ┌──────────────────┐
                │  Express Server  │
                └────────┬─────────┘
                         │
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
 Authentication    Complaint          Tracking /
 Middleware         Controller         Route APIs
        │                │                │
        │        ┌───────┴───────┐        │
        │        ↓               ↓        │
        │   Gemini Vision     EXIF GPS    │
        │        │               │        │
        │        └───────┬───────┘        │
        │                ↓                │
        │          Cloudinary             │
        │                │                │
        └────────────────┼────────────────┘
                         ↓
                  Supabase PostgreSQL
                         │
                         ↓
                    Vehicles Data
```

The documented server-side architecture contains a complaint controller,
driver-tracking controller, and authentication/role middleware.
fileciteturn1file0L62-L65

## 4. Main Backend Modules

### Complaint Controller

`server/controllers/complaintController.js`

Responsibilities:

-   Receive complaint submissions.
-   Process uploaded image buffers.
-   Call Gemini AI Vision.
-   Extract EXIF GPS information.
-   Upload images to Cloudinary.
-   Create/update complaint records in Supabase.
-   Manage complaint status and resolution information.

The supplied project specification identifies this controller as the
core location for AI verification, EXIF extraction, Cloudinary
streaming, and Supabase CRUD operations. fileciteturn1file0L123-L129

### Driver Tracking Controller

`server/controllers/driverTrackingController.js`

Responsibilities:

-   Retrieve vehicle telemetry.
-   Retrieve assigned-task information.
-   Filter vehicle data for the relevant complaint.
-   Provide tracking information to the citizen-facing application.

### Authentication and Authorization Middleware

The backend uses authentication middleware and `verifyAdmin` role
verification for protected administrative operations.

The system distinguishes citizen-facing tracking operations from
admin-only operations.

## 5. Complaint Processing Pipeline

A complaint follows this backend pipeline:

``` text
POST complaint
      ↓
Receive multipart image
      ↓
Gemini AI verification
      ↓
Is valid garbage?
   ┌──┴──┐
   No    Yes
   ↓      ↓
Reject   Extract EXIF GPS
422      ↓
         GPS available?
        ┌──┴──┐
       Yes    No
        ↓      ↓
     Use GPS  Manual map pin
        └──┬───┘
           ↓
    Upload image to Cloudinary
           ↓
   Insert complaint in Supabase
           ↓
      status = Open
```

The documented implementation invokes Gemini, extracts EXIF GPS, uploads
the image to Cloudinary, and inserts the complaint into the Supabase
`complaints` table. fileciteturn1file0L133-L138

## 6. AI Image Verification

The backend uses:

``` text
Google Gemini AI Vision
        ↓
verifyGarbageImage
```

The verification service evaluates an image buffer and returns
information including:

-   `isGarbage`
-   Confidence score
-   Categorization/reason
-   Verification result

A documented validation case expects a non-garbage image to return:

``` text
HTTP 422 Unprocessable Entity
```

with an AI-generated reason, while a valid waste image continues through
the complaint workflow. fileciteturn1file0L107-L110
fileciteturn1file0L158-L161

## 7. EXIF GPS Extraction

The backend attempts to extract embedded GPS information directly from
the uploaded image.

Conceptually:

``` text
Image Buffer
    ↓
EXIF Parser
    ↓
GPS Latitude
GPS Longitude
    ↓
Complaint.latitude
Complaint.longitude
```

If GPS metadata is unavailable, the application can use a manually
selected map location.

The source specification records the location source using:

``` text
EXIF_METADATA
USER_PIN
```

The extraction logic is associated with `imageService.js`.
fileciteturn1file0L20-L24 fileciteturn1file0L107-L110

## 8. Image Upload Architecture

Complaint images and resolution-proof images are stored in
**Cloudinary**.

The backend receives multipart uploads through Multer buffers and
streams them to Cloudinary rather than relying on local disk storage.

``` text
HTTP Multipart Upload
        ↓
Multer Buffer
        ↓
Cloudinary Upload Stream
        ↓
Cloudinary URL
        ↓
Supabase Record
```

This design avoids temporary local file storage and is suitable for
cloud/serverless deployment environments.
fileciteturn1file0L114-L116 fileciteturn1file0L142-L146

## 9. Database

### `complaints`

Important fields:

  Field                  Type / Purpose
  ---------------------- --------------------------------------
  `id`                   UUID / Primary Key
  `citizen_id`           UUID / Citizen reference
  `latitude`             Numeric
  `longitude`            Numeric
  `description`          Text
  `image_url`            Cloudinary URL
  `resolved_image_url`   Resolution proof URL
  `resolution_notes`     Text
  `status`               Open / Assigned / Resolved / Cleaned
  `priority`             Critical / High / Medium / Low
  `category`             Complaint category
  `ai_confidence`        AI confidence value
  `ai_reason`            AI verification explanation
  `gps_source`           EXIF_METADATA / USER_PIN
  `assigned_driver_id`   Foreign key to vehicle
  `created_at`           Timestamp
  `resolved_at`          Timestamp

### `vehicles`

Important fields:

  Field                            Type / Purpose
  -------------------------------- -----------------------------------
  `id / vehicle_id`                UUID / Primary Key
  `driver_name`                    Driver identity
  `license_plate`                  Vehicle registration
  `status`                         Active / In Service / Maintenance
  `speed`                          Current speed
  `capacity_kg`                    Vehicle capacity
  `current_load_kg / payload_kg`   Current load
  `latitude`                       Current latitude
  `longitude`                      Current longitude

These structures and status values are defined in the supplied technical
specification. fileciteturn1file0L71-L103

## 10. Complaint Assignment

The admin workflow uses actual vehicle records from the database rather
than static/mock driver objects.

``` text
Open Complaint
      ↓
Admin Dashboard
      ↓
Query vehicles table
      ↓
Select active vehicle
      ↓
Get vehicles.id UUID
      ↓
Update complaint.assigned_driver_id
      ↓
status = Assigned
```

The documented implementation replaced static driver arrays with live
`vehicles` table queries to prevent assignment/tracking UUID mismatches.
fileciteturn1file0L135-L146

## 11. Vehicle Tracking API Flow

The citizen tracking page requests assigned-driver information using:

``` text
/api/tracking/assigned-drivers?complaintId=...
```

The backend retrieves the relevant vehicle information and returns
telemetry used by the Leaflet tracking interface.

The documented telemetry includes vehicle coordinates and operational
information such as speed/status. fileciteturn1file0L135-L138

### Current Tracking Model

The documented implementation uses database polling rather than
persistent WebSockets.

The specification identifies approximately **3--5 second polling
intervals** as the current approach.

Future production improvement:

``` text
Polling
  ↓
WebSocket / Event-driven telemetry
```

fileciteturn1file0L165-L169

## 12. Resolution Workflow

After the municipal unit clears the reported site:

``` text
Assigned Complaint
       ↓
Cleanup Completed
       ↓
Admin / Driver Uploads Proof
       ↓
Cloudinary
       ↓
resolved_image_url
       ↓
status = Resolved
       ↓
Citizen Feedback
```

The documented workflow stores the resolution image URL and changes the
complaint status to `Resolved`. fileciteturn1file0L133-L138

## 13. Important API / Backend Interfaces

The backend exposes functionality consumed by the React applications
for:

-   Authentication
-   Complaint submission
-   Complaint retrieval
-   Complaint status updates
-   Vehicle administration queries
-   Assigned-driver tracking
-   Resolution proof upload
-   Route/geospatial operations

Keep endpoint paths synchronized with the actual Express router
implementation.

The documented tracking endpoint is:

``` text
GET /api/tracking/assigned-drivers?complaintId=<id>
```

## 14. Environment Variables

Create a `.env` file for local development.

``` env
PORT=3000

SUPABASE_URL=...
SUPABASE_SERVICE_ROLE_KEY=...

CLOUDINARY_CLOUD_NAME=...
CLOUDINARY_API_KEY=...
CLOUDINARY_API_SECRET=...

GEMINI_API_KEY=...
```

Never commit `.env` or service credentials to Git.

The required environment variables are documented in the project
specification. fileciteturn1file0L173-L183

## 15. Getting Started

### Prerequisites

Install:

-   Node.js
-   npm
-   Supabase project
-   Cloudinary account
-   Google Gemini API key

### Installation

``` bash
npm install
```

### Development

``` bash
npm run dev
```

The supplied project specification documents `npm install` and
`npm run dev` as the development setup commands.
fileciteturn1file0L173-L188

## 16. Testing

### AI Image Verification

Test:

``` text
Non-garbage image
        ↓
Gemini verification
        ↓
HTTP 422 + AI reason
```

Valid waste image:

``` text
Waste image
    ↓
AI verification
    ↓
Complaint continues
```

### Vehicle Tracking

Test:

``` text
Assign vehicle
      ↓
Complaint gets vehicle UUID
      ↓
Citizen opens tracking
      ↓
Vehicle marker appears
      ↓
Telemetry card displays active unit
```

These two tests are explicitly documented as working validation
scenarios. fileciteturn1file0L158-L161

## 17. Known Issues and Fixes

### Missing Location Column

**Problem**

``` text
complaints.location does not exist
```

**Solution**

Controllers were updated to use explicit:

``` text
latitude
longitude
```

columns.

### 403 Tracking Error

**Problem**

Admin middleware blocked citizen tracking requests.

**Solution**

Tracking authorization was separated from admin-only authorization or
appropriate token fallback was allowed.

### Leaflet Fullscreen Issue

**Problem**

The Leaflet map could freeze or render incorrectly after fullscreen
changes.

**Solution**

A map recenter/resize helper uses:

``` javascript
map.invalidateSize()
```

when fullscreen state changes.

These fixes are recorded in the project's technical specification.
fileciteturn1file0L150-L154

## 18. Security Considerations

The backend should protect:

-   Supabase service-role credentials
-   Cloudinary API credentials
-   Gemini API key
-   JWT/authentication secrets
-   Administrative routes

Administrative operations should remain protected by role verification
such as `verifyAdmin`.

For production deployment, additional hardening should include rate
limiting, audit logging, stronger token lifecycle management, and
granular municipal roles.

## 19. Future Backend Improvements

The documented future scope includes:

-   IoT smart-bin integration using ultrasonic fill sensors.
-   Automated push notification gateways.
-   WebSocket/event-driven vehicle telemetry.
-   More advanced fleet and route optimization.
-   Production-scale operational analytics.

The current documentation specifically identifies IoT smart dustbins and
automated push notifications as future scope.
fileciteturn1file0L165-L169

## 20. Backend Development Guidelines

When extending the backend:

1.  Keep controllers focused on their respective domain.
2.  Keep authentication and authorization in middleware.
3.  Keep database credentials and API keys in environment variables.
4.  Validate uploaded files before external processing.
5.  Store image binaries in Cloudinary and URLs/metadata in Supabase.
6.  Use actual database UUIDs when linking complaints and vehicles.
7.  Keep API contracts synchronized with the frontend.
8.  Add a test case whenever a new critical workflow is introduced.
9.  Update this README when backend modules, environment variables,
    database fields, or API contracts change.

## Project Status

**Development Status:** Active Development

CivicSync Backend provides the REST API and integration layer for the
citizen and municipal applications, connecting complaint processing, AI
verification, geolocation, cloud image storage, PostgreSQL data, and
municipal vehicle tracking.
