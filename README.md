# Overview

This is a full-stack real-estate rental platform for browsing properties, applying online, and managing rental workflows from both the tenant and property-manager sides.
It extended and updated EdRoh's original tutorial.

## 1. Key Difference

- add a ai chatbot for user to query.
- add unit tests and integration tests for server app, and cover all the endpoints.

## 2. Project Structure

```
client/
   ├──public
   ├──src
   │   ├──app
   │   │   ├──(auth)      //for Cognito-based authentication setup
   │   │   │    └──(auth)
   │   │   ├──(dashboard)
   │   │   │      ├──managers
   │   │   │      │     ├── [id]
   │   │   │      │     │    └──page.tsx
   │   │   │      │     ├── applications
   │   │   │      │     │    └──page.tsx
   │   │   │      │     ├── newProperty
   │   │   │      │     │    └──page.tsx
   │   │   │      │     ├── properties
   │   │   │      │     │    └──page.tsx
   │   │   │      │     ├── settings
   │   │   │      │     │    └──page.tsx
   │   │   │      ├──tenants
   │   │   │      │     ├── applications
   │   │   │      │     │       └──page.tsx
   │   │   │      │     ├── favorites
   │   │   │      │     │       └──page.tsx
   │   │   │      │     ├── residences
   │   │   │      │     │       ├── [id]
   │   │   │      │     │             └──page.tsx
   │   │   │      │     │       └──page.tsx
   │   │   ├──(nondashboard)
   │   │   │      ├──landing
   │   │   │      │     └──page.tsx
   │   │   │      ├──search
   │   │   │      │     └──page.tsx
   │   │   ├──layout.tsx
   │   │   ├──page.tsx
   │   │   ├──providers.tsx
   │   ├──components
   │   ├──hooks
   │   ├──lib
   │   ├──state   //Global search and UI state is managed with Redux, while data fetching is handled through RTK Query.
   │   └──types
   └── package.json

server/
  ├──src/
  │   ├── controllers
  │   ├── middleware
  │   ├── routes
  │   ├── test
  │   │     ├── unit
  │   │     └── integration
  ├── prisma
  │   │   ├── migrations/
  │   │   ├── schema.prisma
  │   │   └── seeddata
  └── package.json

```

## 3. Webpage Structure

```
need no authentication:

- / ----- landing page
- /landing ----- landing page
- /search ----- search page

need authentication: role: manager

- /managers/applications    ----RApprove or deny applications
- /managers/newProperty     ----Create new property listings
- /managers/properties      ----View manager-owned properties
- /managers/properties/:id  ----eview applications
- /managers/settings        ----Settings management


need authentication: role: tenant

- /tenants/applications     ----Rental applications with status display
- /tenants/favorites        ----Favorite properties
- /tenants/residences       ----Current residences
- /tenants/residences/:id
- /tenants/settings         ----Settings management
```

## 4. Server api endpoints Structure

```
no auth:
/api/chat                                  |POST

only for tenants
/tenants                                   |POST
/tenants/:cognitoId                        |GET, PUT
/tenants/:cognitoId/current-residences     |GET
/tenants/:cognitoId/favorites/:propertyId  |POST, DELETE

only for managers
/managers                                  |POST
/managers/:cognitoId                       |GET, PUT
/managers/:cognitoId/properties            |GET

mix auth
/properties                                |GET     |no need
/properties/:id                            |GET     |no need
/properties                                |POST    |manager

mix auth
/applications                              |POST    |tenant
/applications                              |GET     |manager, tenant
/applications/:id/status                   |PUT     |manager

mix auth
/leases                                    |GET     |manager, tenant
/leases/:id/payments                       |GET     |manager, tenant
```

## 5. How to log in:

After setup and fill in environment variables(including AWS Cognito Setting), Users need to sign up and then sign in.

## 6. Minor updates:

## 7. Features:

This application is designed around three user experiences:

1. Public users / prospective tenants can browse listings, search by location, view properties on a map, open a detailed property page, and submit rental applications.
2. Tenants can sign in, save favorites, track applications, and view their current residences.
3. Managers can sign in, create new property listings, review incoming applications, and manage their property portfolio.

#### Public property discovery

- Landing page with a marketing-style homepage and call-to-action sections
- Search page with:
  - location search
  - price filters
  - beds / baths filters
  - property type filters
  - square-footage filters
  - amenities filters
  - grid / list toggle
  - interactive map view using Mapbox
- Property detail page with overview, details, location, image preview, and application entry point

#### Authentication and roles

- AWS Amplify / Cognito-based sign-in and sign-up flow
- User role selection during sign-up (**Tenant** or **Manager**)
- Route-level role gating for tenant and manager API routes

#### Backend rental workflow

- Property CRUD foundation with property creation currently implemented for managers
- Multi-criteria property filtering on the backend
- Geospatial querying using PostGIS coordinates
- Address geocoding during property creation
- Image upload pipeline to Amazon S3
- Lease and payment domain models
- Application creation and status updates

## 8. Tech stack

#### Frontend: client/

- Next.js 15
- React 19
- TypeScript
- Redux Toolkit + RTK Query
- Tailwind CSS
- Radix UI
- React Hook Form + Zod
- Framer Motion
- Mapbox GL JS
- AWS Amplify UI

#### Backend

- Node.js
- Express 5
- TypeScript
- Prisma ORM
- PostgreSQL
- PostGIS
- Multer
- AWS SDK for S3 uploads
- JSON Web Token decoding for role-based route checks

#### Database

- Database: PostgreSQL with PostGIS for geospatial property data

#### Data model

The Prisma schema models the main rental entities:

- `Property`
- `Location`
- `Manager`
- `Tenant`
- `Application`
- `Lease`
- `Payment`

It also supports:

- many-to-many favorites between tenants and properties
- many-to-many tenant/property occupancy relationships
- geographic coordinates stored as `geography(Point, 4326)`

## 9. Notable implementation details

### Search and map experience

The search UI pushes filter state into the URL, syncs it with Redux state, and requests filtered property data from the API. On the backend, the property query supports filtering by favorites, price range, beds, baths, property type, square footage, amenities, availability, and coordinates. Map markers are rendered from the returned property location data.

### Property creation flow

Managers create a property through a multipart form. The backend:

1. receives uploaded images,
2. uploads them to S3,
3. geocodes the address,
4. stores the location with PostGIS coordinates,
5. creates the property record and links it to the manager.

### Application workflow

A tenant can submit an application from a property detail page. The backend creates the application and associated lease data, and managers can later approve or deny applications from their dashboard.

## 10. Local setup

### Prerequisites

- Node.js
- npm
- PostgreSQL
- PostGIS extension enabled in the target database
- AWS Cognito configuration
- Mapbox access token
- Amazon S3 bucket for uploaded property images

#### 1. Clone the repository

```bash
git clone <your-repo-url>
cd real_estate_rentation
```

#### 2. Install dependencies

Install separately for the frontend and backend:

```bash
cd client
npm install

cd ../server
npm install
```

#### 3. Configure environment variables

Create two environment files .env according to .env.example for both apps.
use AWS Cognito to provide relevant env variable.

#### 4. Generate Prisma client and run the database

```bash
cd server
npx prisma generate
npx prisma migrate dev
npm run seed
```

#### 5. Start the backend

```bash
cd server
npm run dev
```

#### 6. Start the frontend

```bash
cd client
npm run dev
```

## 11. About the project

#### Current strengths

- Clear separation between frontend and backend
- Real-world domain modelling for rentals, applications, leases, and payments
- PostGIS-backed location handling
- Role-based tenant/manager user journeys
- Map-based property exploration
- File upload and cloud storage integration

#### Suggested next improvements

- Add API documentation for backend endpoints
- Add screenshots or a short demo GIF
- Add a short architecture diagram
- seed/demo credentials for reviewer access.
- Tighten token verification in the auth middleware if this moves toward production use
- Add deployment instructions for the frontend and backend
