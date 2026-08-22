
# Plate

Plate is a multi-tenant social fitness tracking Progressive Web App (PWA) focused on workout tracking, food logging, analytics, and social accountability.

Users can create accounts, join communities, track workouts, log meals, view progress, create posts, follow other users, like posts, comment on posts, and receive notifications.

Plate uses a microservice-based, event-driven architecture deployed to a Kubernetes cluster running on a home server.

---

## Project Overview

Plate combines the personal tracking features of a fitness journal with the social interaction features of platforms such as Strava and Instagram.

Users can:

* Register and log in
* Create or join a tenant
* Manage a user profile
* Track workouts
* Log meals and nutrition
* View daily, weekly, and monthly summaries
* Create workout, food, progress, and general posts
* View a personalized social feed
* View user profile feeds
* Follow and unfollow other users
* Like and comment on posts
* Receive social notifications
* Configure tenant settings as a tenant administrator

Plate will be delivered as a **Progressive Web App**, allowing users to access it through a browser or install it on supported mobile and desktop devices.

---

## Core Concepts

Plate is built around three primary concepts.

### 1. Fitness Tracking

Users can track:

* Workouts
* Exercises
* Sets
* Repetitions
* Weight
* Duration
* Meals
* Food items
* Calories
* Protein
* Carbohydrates
* Fat
* Fitness progress

### 2. Social Accountability

Users can:

* Share workouts and meals as posts
* Follow other users
* View a personalized feed
* Like posts
* Comment on posts
* View user profiles
* Receive notifications for social activity

### 3. Multi-Tenant Communities

Plate supports tenant-based communities or workspaces.

A tenant can represent:

* A gym
* A sports team
* A fitness group
* A workplace wellness community
* A private training community

Users may belong to one or more tenants, depending on the final membership implementation.

---

## Application Type

Plate will be developed as a:

```text
Progressive Web App (PWA)
```

The PWA approach allows Plate to behave more like a native application while still being delivered through the web.

Planned PWA capabilities include:

* Installable application experience
* Responsive mobile-first design
* Web application manifest
* Service worker support
* Offline-friendly static assets
* Home-screen launch support
* Automatic application updates
* Push notification support in a later phase

---

## Architecture Goals

The Plate architecture is designed around the following goals:

* Clear service ownership
* Independent service deployment
* Loose coupling between business domains
* Reliable event-driven synchronization
* Secure tenant-aware access control
* Independent database ownership
* Scalable stateless services
* Resilient asynchronous processing
* Container orchestration through Kubernetes
* A deployment model that can begin on one server and expand later

---

## Core Architecture Principles

The following rules apply across the Plate system.

### No Direct Domain-Service Communication

Plate domain services must not directly call one another for business-domain synchronization.

For example:

```text
Social Service -> Workout Service
```

should not be required to synchronize workout information.

Instead, services share domain changes through Kafka events.

### Service-Owned Databases

Each service owns its database.

Services must not:

* Query another service's database
* Modify another service's records
* Create cross-service database foreign keys
* Depend on another service's internal schema

Cross-service references are stored as identifiers such as:

```text
userId
tenantId
workoutId
mealId
postId
```

### Event-Driven Synchronization

Cross-service data synchronization occurs through domain events.

Examples include:

```text
UserRegistered
TenantCreated
WorkoutCreated
MealLogged
PostCreated
PostLiked
CommentCreated
UserFollowed
```

### Transactional Outbox

Services do not publish domain events directly to Kafka as part of the main request transaction.

Instead, each service:

1. Begins a database transaction.
2. Writes the business data.
3. Writes an event to an outbox table.
4. Commits both changes together.
5. Returns the API response.
6. Allows Debezium to capture the committed outbox event.
7. Publishes the event to Kafka through Kafka Connect.

This prevents the system from saving business data without publishing the corresponding event.

### Idempotent Consumers

Kafka consumers must safely handle duplicate event delivery.

Consumers store processed event identifiers so the same event is not applied more than once.

### Internal Infrastructure Is Private

The following components must not be directly exposed to the public internet:

* Domain services
* PostgreSQL databases
* Kafka
* Kafka Connect
* Debezium connectors
* Internal Kubernetes Services
* Administrative interfaces

Public traffic enters through Cloudflare and the Kubernetes Ingress.

---

## Technology Stack

### Frontend

```text
Next.js
TypeScript
React
Tailwind CSS
Progressive Web App support
```

The frontend will run as a containerized Next.js application inside the Kubernetes cluster.

The frontend communicates with backend services through the public API Gateway.

---

### Backend

```text
TypeScript
Node.js
REST APIs
Containerized domain services
```

The backend is divided into independently deployed services.

The exact Node.js framework may be selected during implementation.

Possible options include:

* Fastify
* Express
* NestJS
* Hono

---

### Authentication

```text
Token or session-based authentication
Tenant-aware authorization
Role-based access control
```

Better Auth or an equivalent authentication library may be used.

Authentication credentials are validated before trusted user and tenant context is forwarded to backend services.

Backend services must not trust arbitrary identity headers sent directly by a client.

---

### Database

```text
PostgreSQL
Service-owned databases
Transactional outbox tables
Processed-event tables
```

Each domain service owns its persistent data.

The initial deployment may run PostgreSQL workloads inside Kubernetes using persistent storage.

A PostgreSQL operator or externally managed database may be introduced later.

---

### Event Infrastructure

```text
Apache Kafka
Kafka Connect
Debezium
Transactional outbox pattern
Consumer groups
Retry topics
Dead-letter topics
```

Kafka is used for asynchronous domain events and cross-service synchronization.

---

### Deployment and Infrastructure

```text
Ubuntu home server
Docker
Kubernetes
K3s
Nginx Ingress Controller
Cloudflare
GitHub Actions
GitHub Container Registry
PersistentVolumeClaims
Kubernetes ConfigMaps
Kubernetes Secrets
```

Optional or planned infrastructure includes:

```text
Argo CD
Helm
Prometheus
Grafana
Loki
OpenTelemetry
```

---

## Docker and Kubernetes Responsibilities

Docker and Kubernetes serve different purposes in Plate.

```text
Docker builds and packages Plate applications.
GitHub Container Registry stores the Docker images.
Kubernetes runs and manages the containers.
K3s provides the Kubernetes cluster.
Cloudflare provides the public DNS, TLS, and external entry point.
Nginx Ingress routes incoming traffic inside the cluster.
```

Docker Compose may be used for local development, but it is not the target production orchestrator.

---

## Planned Domain Structure

```text
plate.aareid.ca
```

Public Next.js PWA.

```text
api.plate.aareid.ca
```

Public API Gateway endpoint.

Additional subdomains may be used for administrative or infrastructure purposes, but internal domain services should not normally be publicly exposed.

The frontend should send backend requests through:

```text
api.plate.aareid.ca
```

The API Gateway routes requests to the responsible Kubernetes Service.

---

## High-Level Architecture

```text
User Browser / Installed PWA
            |
            v
        Cloudflare
     DNS / TLS / Proxy
            |
            v
  Kubernetes Ingress Resource
            |
            v
   Nginx Ingress Controller
            |
            v
        API Gateway
            |
            +-------------------------------+
            |               |               |
            v               v               v
 Identity & Tenant      Workout         Nutrition
     Service            Service          Service
            |
            +-------------------------------+
            |               |               |
            v               v               v
        Social          Analytics      Notification
        Service          Service          Service
```

Each service owns its database and does not directly query another service's data.

---

## Synchronous Request Flow

User commands and queries follow this general path:

```text
User
-> Cloudflare
-> Kubernetes Ingress
-> Nginx Ingress Controller
-> API Gateway
-> Kubernetes ClusterIP Service
-> Plate Service Pod
-> Service-Owned PostgreSQL Database
```

Synchronous HTTP is used when the user needs an immediate response.

Examples include:

* Registering
* Logging in
* Creating a workout
* Retrieving workout history
* Logging a meal
* Viewing a feed
* Adding a comment

---

## Asynchronous Event Flow

Cross-service synchronization follows this path:

```text
Service Request
-> Business Data and Outbox Event Saved Atomically
-> Transaction Commits
-> Debezium Captures Outbox Record
-> Kafka Connect Publishes Event
-> Kafka Topic
-> Consumer Group
-> Interested Consumer Service
-> Consumer Updates Its Own Database
```

Example:

```text
Workout Service
-> Saves Workout
-> Saves WorkoutCreated Outbox Event
-> Debezium Captures Event
-> Kafka Publishes WorkoutCreated
-> Analytics Service Updates Workout Summary
```

---

## Microservices

Plate is separated into business-focused services.

### 1. API Gateway

The API Gateway is the main backend entry point for the frontend.

Responsibilities:

* Route requests to the responsible service
* Validate authentication context
* Apply common request policies
* Apply rate limits where needed
* Handle CORS
* Attach trusted identity and tenant context
* Hide internal Kubernetes service names
* Provide consistent API responses

Example routes:

```text
/api/auth/*
/api/tenants/*
/api/workouts/*
/api/nutrition/*
/api/social/*
/api/analytics/*
/api/notifications/*
```

The API Gateway is not responsible for domain business logic.

---

### 2. Identity and Tenant Service

The Identity and Tenant Service manages users, authentication, tenant membership, roles, and tenant settings.

Responsibilities:

* User registration
* User login
* User logout
* Session management
* User profiles
* Tenant creation
* Tenant membership
* Tenant invitations
* Tenant role management
* Tenant settings
* Authorization context

Main entities:

```text
users
user_profiles
sessions
tenants
tenant_memberships
tenant_roles
tenant_settings
outbox_events
processed_events
```

Possible roles:

```text
PLATFORM_ADMIN
TENANT_ADMIN
TENANT_MEMBER
```

Possible events produced:

```text
UserRegistered
UserProfileUpdated
TenantCreated
UserJoinedTenant
UserLeftTenant
TenantRoleChanged
TenantSettingsUpdated
```

---

### 3. Workout Service

The Workout Service manages workout tracking.

Responsibilities:

* Create workouts
* Update workouts
* Delete workouts
* Track exercises
* Track workout sets
* Store repetitions and weight
* Store workout duration
* Store workout notes
* Manage workout templates
* Retrieve workout history

Main entities:

```text
workouts
workout_exercises
exercises
set_entries
workout_templates
outbox_events
processed_events
```

Possible events produced:

```text
WorkoutCreated
WorkoutUpdated
WorkoutDeleted
```

External identifiers may include:

```text
user_id
tenant_id
```

These values identify records owned by the Identity and Tenant Service but are not cross-database foreign keys.

---

### 4. Nutrition Service

The Nutrition Service manages meals, food items, nutrition values, and dietary targets.

Responsibilities:

* Create meal logs
* Update meal logs
* Delete meal logs
* Add food items
* Store calories
* Store macronutrients
* Store nutrition targets
* Retrieve food nutrition information
* Integrate with an external nutrition API
* Retrieve nutrition history

Main entities:

```text
meals
meal_entries
food_items
nutrition_facts
daily_nutrition_targets
outbox_events
processed_events
```

Possible events produced:

```text
MealLogged
MealUpdated
MealDeleted
NutritionTargetUpdated
```

The service may call an external nutrition provider to retrieve food information.

It should handle external API failures without corrupting saved meal data.

---

### 5. Social Service

The Social Service manages posts, likes, comments, follows, and social feeds.

Responsibilities:

* Create posts
* Update posts
* Delete posts
* View platform feeds
* View tenant feeds
* View user profile feeds
* Like and unlike posts
* Add comments
* Follow and unfollow users
* Store post visibility
* Maintain social feed projections

Main entities:

```text
posts
comments
post_likes
follows
feed_entries
outbox_events
processed_events
```

Post types:

```text
WORKOUT
MEAL
GENERAL
PROGRESS
```

Post visibility options:

```text
PUBLIC
TENANT_ONLY
FOLLOWERS_ONLY
PRIVATE
```

Possible events produced:

```text
PostCreated
PostUpdated
PostDeleted
PostLiked
PostUnliked
CommentCreated
CommentDeleted
UserFollowed
UserUnfollowed
```

The Social Service may store references such as:

```text
workout_id
meal_id
user_id
tenant_id
```

These are identifiers, not direct database relationships to another service.

---

### 6. Analytics Service

The Analytics Service creates read-optimized fitness and nutrition summaries.

Responsibilities:

* Calculate daily workout summaries
* Calculate weekly workout summaries
* Calculate monthly workout summaries
* Calculate daily nutrition summaries
* Calculate weekly nutrition summaries
* Calculate monthly nutrition summaries
* Track workout trends
* Track nutrition trends
* Calculate progress metrics
* Provide dashboard projections

Main entities:

```text
workout_summaries
nutrition_summaries
progress_metrics
analytics_projections
processed_events
```

The Analytics Service primarily consumes:

```text
WorkoutCreated
WorkoutUpdated
WorkoutDeleted
MealLogged
MealUpdated
MealDeleted
```

Analytics data is derived from events and stored in its own database.

The service does not query the Workout or Nutrition databases directly.

---

### 7. Notification Service

The Notification Service creates and manages user notifications.

Responsibilities:

* Create in-app notifications
* Notify users about new followers
* Notify users about likes
* Notify users about comments
* Store notification preferences
* Mark notifications as read
* Support future email notifications
* Support future push notifications

Main entities:

```text
notifications
notification_preferences
processed_events
```

The Notification Service may consume:

```text
UserFollowed
PostLiked
CommentCreated
TenantInvitationCreated
```

---

## Database Ownership

Each service owns its database or independently managed PostgreSQL schema.

| Service                     | Owned Data                                                       |
| --------------------------- | ---------------------------------------------------------------- |
| Identity and Tenant Service | Users, sessions, profiles, tenants, memberships, roles, settings |
| Workout Service             | Workouts, exercises, sets, templates                             |
| Nutrition Service           | Meals, food items, nutrition facts, nutrition targets            |
| Social Service              | Posts, comments, likes, follows, feeds                           |
| Analytics Service           | Summaries, projections, progress metrics                         |
| Notification Service        | Notifications and notification preferences                       |

Foreign keys are allowed within a service-owned database.

Cross-service foreign keys are prohibited.

---

## Transactional Outbox Pattern

Every service that publishes domain events contains an `outbox_events` table.

A typical outbox event contains:

```text
event_id
aggregate_id
aggregate_type
event_type
tenant_id
payload
schema_version
occurred_at
created_at
```

Example transaction:

```text
BEGIN TRANSACTION

INSERT INTO workouts (...)
INSERT INTO outbox_events (
  event_id,
  aggregate_id,
  aggregate_type,
  event_type,
  tenant_id,
  payload,
  schema_version,
  occurred_at
)

COMMIT TRANSACTION
```

Debezium observes the committed outbox record and publishes the event through Kafka Connect.

---

## Kafka Event Conventions

Possible topic names include:

```text
plate.identity.events
plate.workout.events
plate.nutrition.events
plate.social.events
plate.analytics.events
plate.notification.events
```

Retry and dead-letter topics may use names such as:

```text
plate.workout.events.retry
plate.workout.events.dead-letter
```

Each event should include:

```text
eventId
eventType
aggregateId
tenantId
occurredAt
schemaVersion
correlationId
payload
```

Consumers should use separate consumer groups based on their responsibilities.

Example:

```text
analytics-workout-consumer
social-workout-consumer
notification-social-consumer
```

The final topic strategy may evolve as the system is implemented.

---

## Multi-Tenant Design

Plate is a multi-tenant platform.

A tenant represents a separate organization, community, workspace, or fitness group.

Each tenant may contain:

* Tenant settings
* Tenant administrators
* Tenant members
* Workouts
* Meals
* Posts
* Tenant-specific visibility rules

Users belong to tenants through a membership relationship.

```text
User
-> Tenant Membership
-> Tenant
```

A membership includes information such as:

```text
user_id
tenant_id
role
status
joined_at
```

Tenant-owned records should contain:

```text
tenant_id
```

Backend services are responsible for validating tenant access.

A client must not gain access to another tenant by manually changing a `tenantId` request value.

### Tenant Roles

Possible tenant roles include:

```text
TENANT_ADMIN
TENANT_MEMBER
```

A platform administrator has platform-wide responsibilities and is separate from a tenant administrator.

### Tenant-Aware Constraints

Examples include:

```text
UNIQUE (tenant_id, user_id)
UNIQUE (tenant_id, slug)
UNIQUE (post_id, user_id)
UNIQUE (follower_user_id, following_user_id)
```

---

## Kubernetes Deployment

Plate will run on Kubernetes using K3s.

The initial target environment consists of:

* One Ubuntu home server
* One K3s server node
* One replica for most application services
* Persistent local storage
* Internal Kubernetes networking
* Public access through the Kubernetes Ingress
* Private administration through SSH or Tailscale

A single-node cluster is not highly available.

The initial deployment is intended for learning, development, testing, demonstrations, and early application usage.

---

## Kubernetes Resources

| Component                   | Kubernetes Resource                      | Exposure           | Persistence             |
| --------------------------- | ---------------------------------------- | ------------------ | ----------------------- |
| Next.js PWA                 | Deployment                               | Through Ingress    | No                      |
| API Gateway                 | Deployment                               | Through Ingress    | No                      |
| Identity and Tenant Service | Deployment                               | Internal ClusterIP | No                      |
| Workout Service             | Deployment                               | Internal ClusterIP | No                      |
| Nutrition Service           | Deployment                               | Internal ClusterIP | No                      |
| Social Service              | Deployment                               | Internal ClusterIP | No                      |
| Analytics Service           | Deployment                               | Internal ClusterIP | No                      |
| Notification Service        | Deployment                               | Internal ClusterIP | No                      |
| PostgreSQL                  | StatefulSet or operator-managed resource | Internal           | Yes                     |
| Kafka                       | StatefulSet or operator-managed resource | Internal           | Yes                     |
| Kafka Connect               | Deployment                               | Internal           | Configuration-dependent |
| Debezium Connector          | Kafka Connect connector configuration    | Internal           | No                      |
| Database migrations         | Job                                      | Internal           | No                      |
| Database backups            | CronJob                                  | Internal           | Backup storage          |
| Configuration               | ConfigMap                                | Internal           | No                      |
| Credentials                 | Secret                                   | Internal           | Stored by Kubernetes    |
| Public routing              | Ingress                                  | Public entry point | No                      |

---

## Kubernetes Namespaces

The initial cluster may use a single namespace:

```text
plate
```

As the project grows, workloads may be separated into namespaces such as:

```text
plate-app
plate-data
plate-events
plate-observability
```

The final namespace structure will depend on operational complexity.

---

## Configuration and Secrets

Non-sensitive configuration should use Kubernetes ConfigMaps.

Examples include:

```text
Kafka topic names
Environment names
Feature flags
Logging levels
Internal service URLs
```

Sensitive values should use Kubernetes Secrets.

Examples include:

```text
Database passwords
Authentication secrets
External API keys
Cloudflare credentials
Kafka credentials
```

Secrets must not be committed to GitHub, embedded in Docker images, or stored in ConfigMaps.

---

## Persistent Storage

Persistent components require Kubernetes PersistentVolumeClaims.

Persistent storage is required for:

* PostgreSQL databases
* Kafka data
* Database backups
* Uploaded media when stored locally

Persistent data must survive:

* Pod restarts
* Container restarts
* Application deployments

The initial K3s deployment may use local-path storage.

A more resilient storage provider may be introduced when additional cluster nodes are added.

---

## Public and Internal Networking

The public request flow is:

```text
User
-> Cloudflare
-> Home Server
-> Kubernetes Ingress
-> Nginx Ingress Controller
-> Next.js or API Gateway
```

Only the Ingress should normally be publicly reachable.

The following Kubernetes Services should use internal networking:

```text
ClusterIP
```

Internal components include:

* Domain services
* PostgreSQL
* Kafka
* Kafka Connect
* Analytics consumers
* Notification consumers

---

## Planned Repository Structure

```text
plate/
├── README.md
├── package.json
├── pnpm-workspace.yaml
├── turbo.json
├── .gitignore
├── .env.example
│
├── apps/
│   └── web/
│       ├── package.json
│       ├── next.config.ts
│       ├── public/
│       │   ├── manifest.json
│       │   ├── icons/
│       │   └── screenshots/
│       └── src/
│           ├── app/
│           ├── components/
│           ├── features/
│           ├── hooks/
│           ├── lib/
│           └── types/
│
├── services/
│   ├── api-gateway/
│   ├── identity-tenant-service/
│   ├── workout-service/
│   ├── nutrition-service/
│   ├── social-service/
│   ├── analytics-service/
│   └── notification-service/
│
├── packages/
│   ├── event-contracts/
│   ├── shared-types/
│   ├── validation/
│   └── config/
│
├── infrastructure/
│   ├── docker/
│   │   └── compose.local.yml
│   ├── kubernetes/
│   │   ├── namespaces/
│   │   ├── ingress/
│   │   ├── applications/
│   │   ├── databases/
│   │   ├── kafka/
│   │   ├── config/
│   │   ├── secrets/
│   │   ├── jobs/
│   │   └── storage/
│   ├── helm/
│   └── argocd/
│
├── docs/
│   ├── PLATE_SYSTEM_ARCHITECTURE.md
│   ├── api/
│   ├── events/
│   └── diagrams/
│       ├── use-case.puml
│       ├── architecture.puml
│       ├── deployment.puml
│       ├── class-diagram.puml
│       ├── erd/
│       ├── activity-diagrams/
│       └── sequence-diagrams/
│
└── .github/
    └── workflows/
        ├── test.yml
        ├── build-images.yml
        └── deploy.yml
```

This structure may change as the implementation evolves.

---

## Example API Endpoints

### Authentication and Tenant Routes

```text
POST   /api/auth/register
POST   /api/auth/login
POST   /api/auth/logout
GET    /api/auth/me

PATCH  /api/users/me/profile

POST   /api/tenants
GET    /api/tenants/:tenantId
POST   /api/tenants/:tenantId/members
PATCH  /api/tenants/:tenantId/settings
PATCH  /api/tenants/:tenantId/members/:userId/role
```

### Workout Routes

```text
POST   /api/workouts
GET    /api/workouts/:workoutId
PATCH  /api/workouts/:workoutId
DELETE /api/workouts/:workoutId

GET    /api/users/:userId/workouts
GET    /api/tenants/:tenantId/workouts
```

### Nutrition Routes

```text
POST   /api/nutrition/meals
GET    /api/nutrition/meals/:mealId
PATCH  /api/nutrition/meals/:mealId
DELETE /api/nutrition/meals/:mealId

GET    /api/nutrition/foods/search
GET    /api/users/:userId/meals
GET    /api/users/:userId/nutrition-summary
```

### Social Routes

```text
POST   /api/social/posts
GET    /api/social/feed
GET    /api/social/users/:userId/feed

POST   /api/social/posts/:postId/likes
DELETE /api/social/posts/:postId/likes

POST   /api/social/posts/:postId/comments
GET    /api/social/posts/:postId/comments

POST   /api/social/users/:userId/follow
DELETE /api/social/users/:userId/follow
```

### Analytics Routes

```text
GET /api/analytics/workouts/daily
GET /api/analytics/workouts/weekly
GET /api/analytics/workouts/monthly

GET /api/analytics/nutrition/daily
GET /api/analytics/nutrition/weekly
GET /api/analytics/nutrition/monthly

GET /api/analytics/progress
```

### Notification Routes

```text
GET   /api/notifications
PATCH /api/notifications/:notificationId/read
PATCH /api/notifications/read-all
GET   /api/notifications/preferences
PATCH /api/notifications/preferences
```

---

## Core Workflows

### User Creates an Account

```text
User opens Plate
-> User submits registration details
-> API Gateway validates the request
-> Identity and Tenant Service validates registration data
-> User and profile records are created
-> UserRegistered event is written to the outbox
-> Database transaction commits
-> Authentication session or token is returned
-> Debezium captures UserRegistered
-> Kafka publishes the event
-> Interested consumers process the event
-> User is redirected to the dashboard
```

---

### User Logs a Workout

```text
User submits workout
-> API Gateway validates authentication
-> Workout Service validates tenant access
-> Workout Service validates workout data
-> Workout and exercises are saved
-> WorkoutCreated event is written to the outbox
-> Database transaction commits
-> API returns the saved workout
-> Debezium captures WorkoutCreated
-> Kafka publishes the event
-> Analytics Service updates workout summaries
```

---

### User Creates a Workout Post

Creating a workout and creating a post are separate commands.

```text
User logs workout
-> Workout Service saves workout
-> WorkoutCreated event is published

User chooses to share workout
-> User submits post request containing workoutId
-> Social Service creates the post
-> PostCreated event is written to the Social Service outbox
-> Post appears in relevant feeds
```

The Social Service does not directly query the Workout database.

It may store a snapshot or projection of the workout details required to display the post.

---

### User Logs a Meal

```text
User searches for food
-> Nutrition Service requests nutrition information from external API
-> Nutrition Service returns food options
-> User submits meal
-> Nutrition Service saves meal and food entries
-> MealLogged event is written to the outbox
-> Transaction commits
-> Debezium captures MealLogged
-> Kafka publishes the event
-> Analytics Service updates nutrition summaries
```

---

### User Follows Another User

```text
User submits follow request
-> API Gateway validates authentication
-> Social Service validates the request
-> Social Service prevents self-following and duplicate follows
-> Follow relationship is created
-> UserFollowed event is written to the outbox
-> Transaction commits
-> Debezium captures UserFollowed
-> Kafka publishes the event
-> Notification Service creates a notification
```

---

### Tenant Administrator Configures Tenant Settings

```text
Tenant administrator opens settings
-> API Gateway validates authentication
-> Identity and Tenant Service verifies tenant membership
-> Identity and Tenant Service verifies TENANT_ADMIN role
-> Settings are validated
-> Tenant settings are updated
-> TenantSettingsUpdated event is written to the outbox
-> Transaction commits
-> Updated settings are returned
```

---

## Planned Data Model

### Identity and Tenant Database

```text
users
- id
- email
- username
- password_hash
- status
- created_at
- updated_at

user_profiles
- id
- user_id
- display_name
- bio
- profile_image_url
- fitness_goal
- updated_at

sessions
- id
- user_id
- token_hash
- expires_at
- created_at

tenants
- id
- name
- slug
- owner_user_id
- status
- created_at
- updated_at

tenant_memberships
- id
- tenant_id
- user_id
- role
- status
- joined_at

tenant_settings
- id
- tenant_id
- default_post_visibility
- allow_public_discovery
- allow_comments
- updated_at

outbox_events
- event_id
- aggregate_id
- aggregate_type
- event_type
- tenant_id
- payload
- schema_version
- occurred_at
```

---

### Workout Database

```text
workouts
- id
- tenant_id
- user_id
- title
- workout_type
- duration_minutes
- notes
- started_at
- completed_at
- created_at
- updated_at

exercises
- id
- name
- muscle_group
- description

workout_exercises
- id
- workout_id
- exercise_id
- sequence_number
- notes

set_entries
- id
- workout_exercise_id
- set_number
- reps
- weight
- duration_seconds
- rest_seconds

workout_templates
- id
- tenant_id
- user_id
- name
- description
- created_at

outbox_events
- event_id
- aggregate_id
- aggregate_type
- event_type
- tenant_id
- payload
- schema_version
- occurred_at

processed_events
- event_id
- consumer_name
- processed_at
```

---

### Nutrition Database

```text
meals
- id
- tenant_id
- user_id
- meal_type
- title
- notes
- logged_at
- created_at
- updated_at

meal_entries
- id
- meal_id
- food_item_id
- quantity
- unit
- calories
- protein
- carbohydrates
- fat

food_items
- id
- external_source
- external_id
- name
- brand
- serving_size
- serving_unit

nutrition_facts
- id
- food_item_id
- calories
- protein
- carbohydrates
- fat
- fibre
- sugar
- sodium

daily_nutrition_targets
- id
- tenant_id
- user_id
- calorie_target
- protein_target
- carbohydrate_target
- fat_target
- effective_date

outbox_events
- event_id
- aggregate_id
- aggregate_type
- event_type
- tenant_id
- payload
- schema_version
- occurred_at

processed_events
- event_id
- consumer_name
- processed_at
```

---

### Social Database

```text
posts
- id
- tenant_id
- user_id
- post_type
- referenced_workout_id
- referenced_meal_id
- caption
- media_url
- visibility
- created_at
- updated_at

comments
- id
- post_id
- user_id
- body
- created_at
- updated_at

post_likes
- id
- post_id
- user_id
- created_at

follows
- id
- follower_user_id
- followed_user_id
- created_at

feed_entries
- id
- user_id
- post_id
- score
- created_at

outbox_events
- event_id
- aggregate_id
- aggregate_type
- event_type
- tenant_id
- payload
- schema_version
- occurred_at

processed_events
- event_id
- consumer_name
- processed_at
```

---

### Analytics Database

```text
workout_summaries
- id
- tenant_id
- user_id
- period_type
- period_start
- workout_count
- total_duration
- total_volume
- updated_at

nutrition_summaries
- id
- tenant_id
- user_id
- period_type
- period_start
- calories
- protein
- carbohydrates
- fat
- updated_at

progress_metrics
- id
- tenant_id
- user_id
- metric_type
- metric_value
- measured_at

processed_events
- event_id
- consumer_name
- processed_at
```

---

### Notification Database

```text
notifications
- id
- tenant_id
- recipient_user_id
- actor_user_id
- notification_type
- reference_id
- message
- read_at
- created_at

notification_preferences
- id
- user_id
- likes_enabled
- comments_enabled
- follows_enabled
- push_enabled
- email_enabled
- updated_at

processed_events
- event_id
- consumer_name
- processed_at
```

---

## CI/CD and Deployment Flow

The intended deployment flow is:

```text
Developer
-> Git Push
-> GitHub Repository
-> GitHub Actions
-> Automated Tests
-> Docker Image Build
-> GitHub Container Registry
-> Kubernetes Manifest or Helm Update
-> K3s Pulls New Image
-> Kubernetes Rolling Deployment
```

If Argo CD is introduced, the flow becomes:

```text
GitHub Repository
-> Argo CD Detects Desired-State Change
-> Argo CD Synchronizes K3s Cluster
-> Kubernetes Performs Rolling Deployment
```

Argo CD should be described as planned until it is configured in the repository.

---

## Development Phases

### Phase 1: Project Foundation

* Set up monorepo
* Set up Next.js TypeScript application
* Add PWA configuration
* Create initial service projects
* Add Dockerfiles
* Configure local development
* Create shared code packages
* Create architecture diagrams

### Phase 2: Kubernetes Foundation

* Install and configure K3s
* Create Plate namespace
* Configure Nginx Ingress Controller
* Configure Cloudflare routing
* Configure Kubernetes Secrets and ConfigMaps
* Create initial Deployments and Services
* Configure persistent storage

### Phase 3: Authentication and Tenants

* Register users
* Log in users
* Create sessions
* Create tenants
* Add tenant memberships
* Add tenant roles
* Add user profiles
* Add tenant settings

### Phase 4: Workout Tracking

* Create workout logs
* Add exercises
* Add workout sets
* Add workout templates
* Retrieve workout history
* Publish workout domain events

### Phase 5: Nutrition Logging

* Integrate external nutrition API
* Create meal logs
* Add meal entries
* Track calories and macros
* Add nutrition targets
* Publish nutrition domain events

### Phase 6: Event Infrastructure

* Deploy Kafka
* Deploy Kafka Connect
* Configure Debezium
* Add outbox tables
* Configure Kafka topics
* Add consumer groups
* Add retry handling
* Add dead-letter topics
* Add processed-event tracking

### Phase 7: Social Platform

* Create posts
* Display feeds
* Display user profile feeds
* Add likes
* Add comments
* Add follows
* Publish social domain events

### Phase 8: Analytics and Notifications

* Create workout summaries
* Create nutrition summaries
* Create progress metrics
* Add in-app notifications
* Add notification preferences

### Phase 9: PWA Polish

* Add application manifest
* Add service worker
* Improve mobile responsiveness
* Add install support
* Improve offline asset handling
* Add loading and error states

### Phase 10: CI/CD and Observability

* Add GitHub Actions
* Push images to GitHub Container Registry
* Add automated Kubernetes deployments
* Add readiness and liveness probes
* Add centralized logging
* Add metrics and monitoring
* Add backup automation

---

## Failure Handling

Plate should account for the following failure scenarios.

### Duplicate Events

Consumers use processed-event records to avoid applying the same event more than once.

### Consumer Failures

Failed event processing may use:

* Retry topics
* Delayed retries
* Dead-letter topics
* Error logging
* Manual replay tooling

### External Nutrition API Failure

The Nutrition Service should:

* Return a clear error when live nutrition lookup is unavailable
* Allow retrying the lookup
* Avoid saving incomplete food records
* Use cached nutrition data where appropriate

### Pod Failure

Kubernetes restarts failed Pods.

Readiness probes prevent unhealthy Pods from receiving traffic.

Liveness probes restart Pods that become unresponsive.

### Kafka Downtime

Business requests can continue saving domain data and outbox events while Kafka or Kafka Connect is temporarily unavailable.

The unpublished outbox records remain in PostgreSQL until the event infrastructure recovers.

### Database Failure

The service should fail the transaction and return an error without partially saving business data or outbox events.

### Failed Deployment

Kubernetes rolling updates and readiness checks reduce the chance of sending traffic to an unhealthy version.

Rollback procedures will be documented as deployment automation is introduced.

---

## Security Boundaries

The Plate security model includes:

* Cloudflare-managed public DNS and TLS
* Public traffic through Kubernetes Ingress
* Internal ClusterIP Services
* Private PostgreSQL databases
* Private Kafka infrastructure
* Kubernetes Secrets for credentials
* Backend tenant authorization
* Trusted authentication context
* Minimum container privileges
* No secrets embedded in Docker images
* Private server administration through SSH or Tailscale

Tenant access must be enforced by backend services, not only by frontend navigation.

---

## Observability

Planned observability capabilities include:

* Structured application logs
* Kubernetes Pod logs
* Health endpoints
* Readiness probes
* Liveness probes
* Service metrics
* Kafka consumer lag monitoring
* Database health monitoring
* Failed event monitoring
* Distributed tracing
* Deployment status monitoring

Possible tools include:

```text
Prometheus
Grafana
Loki
OpenTelemetry
```

These tools should only be described as implemented once they exist in the repository or cluster.

---

## Future Improvements

Possible future capabilities include:

* Push notifications
* Email notifications
* Workout streaks
* Progress charts
* Media uploads
* Barcode scanning
* Wearable integration
* Fitness challenges
* Leaderboards
* User search
* Tenant search
* Private accounts
* Content moderation
* AI-generated workout summaries
* AI-assisted meal analysis
* Horizontal Pod Autoscaling
* Additional K3s worker nodes
* Replicated Kafka
* Highly available PostgreSQL
* Distributed persistent storage
* Automated off-site backups
* Advanced secret management

---

## Architecture Documentation

Detailed system architecture documentation is located at:

```text
docs/PLATE_SYSTEM_ARCHITECTURE.md
```

Architecture diagrams are located under:

```text
docs/diagrams/
```

The diagrams include:

* Use case diagram
* Activity diagrams
* Class diagram
* Service ERDs
* System architecture diagram
* Kubernetes deployment diagram
* Event-flow or sequence diagrams

---

## Architecture Summary

Plate is a multi-tenant, microservice-based social fitness PWA.

The target architecture uses:

```text
Next.js and TypeScript
Node.js domain services
PostgreSQL service-owned databases
Transactional outbox pattern
Debezium
Kafka Connect
Apache Kafka
Docker
Kubernetes
K3s
Nginx Ingress Controller
Cloudflare
GitHub Actions
GitHub Container Registry
```

The primary domain services are:

```text
Identity and Tenant Service
Workout Service
Nutrition Service
Social Service
Analytics Service
Notification Service
```

Plate services do not directly communicate with one another for domain synchronization.

Each service writes its domain data and an outbox event within the same database transaction. Debezium captures the event and publishes it to Kafka, where interested services consume it and update their own data independently.

The initial deployment will run on a single Ubuntu home server using K3s, with an architecture that can later expand to additional Kubernetes worker nodes and more resilient infrastructure.
