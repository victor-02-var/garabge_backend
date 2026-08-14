# CivicSync -- Admin Dashboard

The **CivicSync Admin Dashboard** is the municipal operations interface
for managing citizen waste complaints, assigning collection vehicles,
monitoring field operations, uploading resolution proof, and planning
municipal routes.

It forms the administrative side of the CivicSync platform and works
with the CivicSync backend and Supabase data layer.

> **Application:** CivicSync Admin Dashboard\
> **Frontend:** React + Vite\
> **Maps:** Leaflet / react-leaflet\
> **Backend:** Node.js + Express.js\
> **Database:** Supabase PostgreSQL

------------------------------------------------------------------------

## 1. Overview

The Admin Dashboard is designed for municipal staff who need to manage
the complete operational lifecycle of waste-related complaints.

The main administrative workflow is:

``` text
Citizen Complaint
       ↓
Admin Grievance Dashboard
       ↓
Review / Triage
       ↓
Vehicle Assignment
       ↓
Field Vehicle Tracking
       ↓
Cleanup
       ↓
Resolution Proof Upload
       ↓
Complaint Resolved
```

The documented CivicSync system includes an admin grievance table for
triage, filtering, vehicle assignment, and resolution-proof upload,
together with a route optimizer for territory/geofence mapping and fleet
route planning. fileciteturn1file0L53-L65

------------------------------------------------------------------------

## 2. Key Features

### 2.1 Grievance Management

The **Grievances** module provides the administrative interface for
managing incoming citizen complaints.

It supports:

-   Viewing complaints
-   Complaint triage
-   Filtering complaints
-   Reviewing complaint information
-   Assigning municipal vehicles/drivers
-   Updating complaint status
-   Uploading resolution proof

The documented project workflow starts with complaints stored with
`Open` status and moves them to `Assigned` after administrative vehicle
assignment. fileciteturn1file0L29-L37

### 2.2 Vehicle Assignment

Administrators can assign an active municipal vehicle to a complaint.

The assignment flow is:

``` text
Open Complaint
      ↓
Admin opens assignment interface
      ↓
Fetch vehicles from database
      ↓
Select active vehicle
      ↓
Store vehicles.id
      ↓
complaint.assigned_driver_id
      ↓
Status → Assigned
```

The implementation intentionally uses real vehicle UUIDs from the
`vehicles` database table instead of static/mock driver arrays. This
prevents mismatches between vehicle assignment and citizen tracking.
fileciteturn1file0L135-L146

### 2.3 Resolution Proof

After the assigned field unit completes the cleanup, the administrative
interface can upload an **After** image.

The workflow is:

``` text
Assigned Complaint
       ↓
Cleanup Completed
       ↓
Upload Resolution Proof
       ↓
Cloudinary
       ↓
resolved_image_url
       ↓
Complaint Status → Resolved
```

The resolution image is stored externally while the complaint record
retains the image URL and resolution metadata.
fileciteturn1file0L31-L37

### 2.4 Route Optimizer

The `RouteOptimizer` module provides:

-   Territory/geofence visualization
-   Local-area/ward mapping
-   OpenStreetMap geocoding
-   Fleet route-planning functionality
-   Fullscreen map interaction

The project specification identifies territory geofencing and
OpenStreetMap/Nominatim-based geospatial functionality as part of the
CivicSync platform. fileciteturn1file0L20-L24

### 2.5 Fleet Visibility

The Admin Dashboard works with the vehicle data used by the tracking
system.

Vehicle information documented by the project includes:

-   Driver name
-   License plate
-   Vehicle status
-   Speed
-   Capacity
-   Current load
-   Latitude
-   Longitude

These values are stored in the `vehicles` table.
fileciteturn1file0L93-L103

------------------------------------------------------------------------

## 3. Technology Stack

  Layer                   Technology
  ----------------------- -------------------------
  Frontend                React
  Build Tool              Vite
  Routing                 React Router
  Icons                   Lucide React
  Maps                    Leaflet
  React Map Integration   react-leaflet
  Styling                 Custom CSS modules
  Backend                 Node.js + Express.js
  Database                Supabase PostgreSQL
  Image Storage           Cloudinary
  AI Verification         Google Gemini AI Vision
  Geocoding               OpenStreetMap Nominatim

The documented CivicSync stack uses React/Vite on the client side,
Leaflet/react-leaflet for maps, Node.js/Express.js on the server,
Supabase PostgreSQL for data, Cloudinary for image storage, Gemini
Vision for AI verification, and Nominatim for geocoding.
fileciteturn1file0L41-L49

------------------------------------------------------------------------

## 4. Admin Dashboard Modules

### `Grievances.jsx`

The primary municipal grievance-management module.

Responsibilities:

-   Display complaint records
-   Filter and triage complaints
-   Open assignment interface
-   Retrieve real vehicle records
-   Assign vehicles
-   Upload resolution proof
-   Update complaint lifecycle

The source specification explicitly identifies
`src/pages/Grievances.jsx` as the admin management table for triage,
driver assignment, and resolution-proof upload.
fileciteturn1file0L121-L129

### `RouteOptimizer.jsx`

The route-planning and territory-management interface.

Responsibilities:

-   Display municipal territories
-   Configure/view geofenced areas
-   Use OpenStreetMap geocoding
-   Support fleet route planning
-   Provide fullscreen map interaction

The project documentation identifies this page as responsible for
territory geofencing, OpenStreetMap geocoding, and fleet route planning.
fileciteturn1file0L123-L129

------------------------------------------------------------------------

## 5. Complaint Lifecycle

The Admin Dashboard operates on the complaint lifecycle:

``` text
OPEN
  │
  │ Admin reviews
  ↓
ASSIGNED
  │
  │ Vehicle performs cleanup
  ↓
RESOLVED
```

The database also documents `Cleaned` as a possible complaint status.

Important complaint fields include:

``` text
id
citizen_id
latitude
longitude
description
image_url
resolved_image_url
resolution_notes
status
priority
category
ai_confidence
ai_reason
gps_source
assigned_driver_id
created_at
resolved_at
```

These fields are defined in the supplied project specification.
fileciteturn1file0L73-L91

------------------------------------------------------------------------

## 6. Admin Assignment Logic

A key implementation decision was replacing static/mock drivers with
live database vehicle records.

### Previous approach

``` text
Static Driver Array
       ↓
Admin Assignment
       ↓
Potential ID mismatch
       ↓
Tracking problems
```

### Current approach

``` text
Supabase vehicles table
       ↓
Fetch actual vehicles
       ↓
Admin selects vehicle
       ↓
vehicles.id
       ↓
complaints.assigned_driver_id
       ↓
Tracking uses same vehicle
```

This ensures that the vehicle assigned by the administrator is the same
vehicle referenced by the citizen tracking workflow.
fileciteturn1file0L142-L145

------------------------------------------------------------------------

## 7. Resolution Proof Workflow

The resolution workflow provides evidence that a reported site has been
cleaned.

``` text
Complaint = Assigned
       ↓
Field work completed
       ↓
Admin uploads "After" image
       ↓
Multer receives multipart data
       ↓
Cloudinary upload
       ↓
Cloudinary URL returned
       ↓
Supabase complaint update
       ↓
resolved_image_url saved
       ↓
status = Resolved
```

Cloudinary stream uploads are used instead of local disk storage, which
keeps uploaded image processing compatible with cloud/serverless
environments. fileciteturn1file0L142-L146

------------------------------------------------------------------------

## 8. Route Optimizer

The Route Optimizer is the geospatial planning module of the
administrative interface.

### Core workflow

``` text
Municipal Territory
       ↓
Geofence / Boundary
       ↓
OpenStreetMap Geocoding
       ↓
Location Data
       ↓
Fleet Route Planning
       ↓
Map Visualization
```

The documented system includes territory boundary assignment,
OpenStreetMap geocoding for local wards, and route planning.
fileciteturn1file0L20-L24

### Mapping

The application uses:

-   Leaflet
-   react-leaflet
-   OpenStreetMap/Nominatim geocoding

The route optimizer also supports fullscreen map interaction.

------------------------------------------------------------------------

## 9. Backend Integration

The Admin Dashboard communicates with the CivicSync Express REST API.

The backend is responsible for:

-   Authentication
-   Administrative authorization
-   Complaint CRUD operations
-   Vehicle queries
-   Vehicle assignment
-   Resolution updates
-   Image uploads
-   Fleet tracking data
-   Geospatial operations

The backend contains a complaint controller, driver-tracking controller,
and authentication/role middleware including `verifyAdmin`.
fileciteturn1file0L62-L65

### Central API Layer

The frontend uses the centralized API service:

``` text
src/services/api.js
```

The documented API wrapper handles:

-   JWT token injection
-   JSON requests
-   Multipart form-data requests
-   Authentication-aware requests

fileciteturn1file0L114-L116

------------------------------------------------------------------------

## 10. Authentication & Authorization

Administrative operations must be protected from normal citizen access.

The documented backend uses:

``` text
Authentication
      ↓
Role Verification
      ↓
verifyAdmin
      ↓
Administrative Operations
```

Admin-only operations include functionality such as:

-   Complaint triage
-   Vehicle assignment
-   Resolution proof management
-   Administrative fleet operations

Citizen tracking routes need to remain separately accessible according
to their intended authorization flow. The project documentation records
a previous `403 Forbidden` issue caused by admin middleware blocking
citizen tracking requests, which was resolved by separating public
tracking authorization from admin-only authorization.
fileciteturn1file0L150-L154

------------------------------------------------------------------------

## 11. Database Integration

The Admin Dashboard primarily works with complaint and vehicle records.

### `complaints`

``` text
complaints
├── id
├── citizen_id
├── latitude
├── longitude
├── description
├── image_url
├── resolved_image_url
├── resolution_notes
├── status
├── priority
├── category
├── ai_confidence
├── ai_reason
├── gps_source
├── assigned_driver_id
├── created_at
└── resolved_at
```

### `vehicles`

``` text
vehicles
├── id / vehicle_id
├── driver_name
├── license_plate
├── status
├── speed
├── capacity_kg
├── current_load_kg / payload_kg
├── latitude
└── longitude
```

The database structure is defined in the project specification.
fileciteturn1file0L71-L103

------------------------------------------------------------------------

## 12. Important Technical Decisions

### Real Vehicle UUID Mapping

Live `vehicles.id` values are used for assignment rather than mock
identifiers.

**Reason:** prevents assignment/tracking synchronization errors.

### Cloudinary Stream Upload

Resolution images are streamed to Cloudinary.

**Reason:** avoids unnecessary local file storage and supports
cloud/serverless deployment.

### Explicit Latitude/Longitude

The complaint model uses:

``` text
latitude
longitude
```

instead of a generic `location` column.

**Reason:** resolves the documented database error:

``` text
complaints.location does not exist
```

The controllers were updated to use explicit latitude/longitude fields.
fileciteturn1file0L150-L153

### Leaflet Map Resize Handling

Fullscreen changes require Leaflet map size recalculation.

The implementation uses:

``` javascript
map.invalidateSize()
```

to prevent map rendering/freezing issues after fullscreen transitions.
fileciteturn1file0L152-L154

------------------------------------------------------------------------

## 13. Getting Started

### Prerequisites

Install:

-   Node.js
-   npm
-   Modern web browser
-   Access to the CivicSync backend
-   Supabase project configured by the backend

### Installation

``` bash
git clone <repository-url>
cd CivicSync-Admin-Dashboard
npm install
```

### Development

``` bash
npm run dev
```

The exact repository name/path may differ from the local project. Update
the command if the actual repository directory uses a different name.

------------------------------------------------------------------------

## 14. Environment Configuration

The Admin Dashboard should use the project's configured frontend
environment variables for backend/API configuration.

Do not place backend secrets directly in frontend source code.

**Never expose:**

``` text
SUPABASE_SERVICE_ROLE_KEY
CLOUDINARY_API_SECRET
GEMINI_API_KEY
JWT signing secrets
```

These belong to the backend environment.

The backend's documented environment configuration includes:

``` env
PORT=3000
SUPABASE_URL=...
SUPABASE_SERVICE_ROLE_KEY=...
CLOUDINARY_CLOUD_NAME=...
CLOUDINARY_API_KEY=...
CLOUDINARY_API_SECRET=...
GEMINI_API_KEY=...
```

fileciteturn1file0L173-L183

------------------------------------------------------------------------

## 15. Testing

### Complaint Triage

``` text
Open complaint
      ↓
Admin opens Grievances
      ↓
Complaint details displayed
      ↓
Admin filters/reviews
```

### Vehicle Assignment

``` text
Open complaint
      ↓
Assignment interface
      ↓
Vehicles loaded from database
      ↓
Admin selects vehicle
      ↓
assigned_driver_id updated
      ↓
status = Assigned
```

### Resolution Proof

``` text
Assigned complaint
      ↓
Upload after-cleanup image
      ↓
Cloudinary upload
      ↓
resolved_image_url saved
      ↓
status = Resolved
```

### Tracking Integration

``` text
Assigned vehicle
      ↓
Citizen opens tracking
      ↓
Tracking API retrieves assigned unit
      ↓
Leaflet displays vehicle marker
```

The project specification explicitly documents vehicle assignment and
live tracking validation. fileciteturn1file0L135-L138

------------------------------------------------------------------------

## 16. Known Issues and Fixes

### `complaints.location` Does Not Exist

**Cause:** Backend queried a nonexistent `location` column.

**Fix:** Use:

``` text
latitude
longitude
```

### Tracking Endpoint Returns 403

**Cause:** Citizen tracking request was blocked by admin middleware.

**Fix:** Separate citizen tracking authorization from admin-only routes.

### Leaflet Freezes in Fullscreen

**Cause:** Leaflet does not automatically recalculate its container
dimensions after fullscreen changes.

**Fix:**

``` javascript
map.invalidateSize()
```

on fullscreen state changes.

These issues and fixes are documented in the supplied technical
specification. fileciteturn1file0L150-L154

------------------------------------------------------------------------

## 17. Current Limitations

The current documented system has several limitations:

-   Fleet telemetry uses database polling rather than persistent
    WebSockets.
-   The documented project does not establish physical smart-bin
    hardware integration.
-   Route optimization is currently represented through the
    route-planning/geospatial interface; city-scale optimization
    performance is not documented.
-   External services such as Supabase, Cloudinary, Gemini, and
    OpenStreetMap/Nominatim introduce availability and rate-limit
    dependencies.

The documented current/future scope specifically identifies database
polling as the current tracking approach and IoT smart bins as future
work. fileciteturn1file0L165-L169

------------------------------------------------------------------------

## 18. Future Improvements

Potential improvements for the Admin Dashboard include:

-   Real-time WebSocket fleet telemetry
-   Advanced route optimization using vehicle capacity and service
    constraints
-   Traffic-aware routing
-   Automated assignment based on distance and vehicle capacity
-   Fleet utilization analytics
-   Complaint heatmaps
-   Ward-level performance dashboards
-   SLA monitoring
-   Automated notifications
-   IoT smart-bin integration
-   Historical operational analytics
-   Audit logs for administrative actions

The documented future scope specifically includes IoT smart dustbins and
automated push-notification gateways. fileciteturn1file0L165-L169

------------------------------------------------------------------------

## 19. Development Guidelines

When modifying the Admin Dashboard:

1.  Keep API communication centralized through `src/services/api.js`.
2.  Use actual database vehicle UUIDs for assignments.
3.  Do not hard-code production credentials.
4.  Keep admin-only functionality behind appropriate authorization.
5.  Keep complaint status transitions consistent with backend logic.
6.  Validate uploaded resolution images before submission.
7.  Keep Leaflet map resizing logic when changing fullscreen/map
    layouts.
8.  Keep route/geospatial functionality synchronized with the backend
    API.
9.  Add a test case for every major workflow change.
10. Update this README when modules, APIs, database fields, or workflows
    change.

------------------------------------------------------------------------

## 20. Project Status

**Development Status:** Active Development

The CivicSync Admin Dashboard provides the municipal-facing operational
interface for grievance management, vehicle assignment, fleet
visibility, resolution-proof management, and route/geospatial planning.

It works as part of the larger CivicSync platform alongside the Citizen
Dashboard and Express backend.
