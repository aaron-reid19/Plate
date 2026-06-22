# Plate

Plate is a multi-tenant social fitness tracking Progressive Web App (PWA) focused on workout tracking, food logging, and social accountability. Users can create an account, log meals, track workouts, create posts, follow other users, like posts, comment on posts, and view user profile feeds.

The platform is designed with a microservice architecture where each major business domain is separated into its own API service and database.

---

## Project Overview

Plate combines the personal tracking features of a fitness journal with the social interaction features of platforms like Strava and Instagram.

Users can:

- Register and log in
- Create or join a tenant account
- Track workouts
- Log food and meals
- Create workout and food posts
- View a social feed
- View user profile feeds
- Follow other users
- Like and comment on posts
- Configure tenant settings as a tenant admin

This application will be built as a **Progressive Web App (PWA)** so users can install it on supported mobile and desktop devices.

---

## Core Concept

Plate is built around two main ideas:

1. **Fitness Tracking**

   - Users can track workouts, exercises, sets, reps, weight, meals, calories, macros, and progress.
2. **Social Accountability**

   - Users can share workouts and food logs as posts, interact with other users, and build a feed based on people they follow.

---

## Application Type

Plate will be developed as a:

```text
Progressive Web App (PWA)
```

The PWA approach allows the app to behave more like a native mobile app while still being delivered through the web.

Planned PWA features include:

- Installable app experience
- Responsive mobile-first design
- App manifest
- Service worker support
- Offline-friendly static assets
- Home screen launch support
- Automatic updates when the deployed web app changes

---

## Tech Stack

### Frontend

```text
Next.js
TypeScript
React
Tailwind CSS
PWA support
```

The frontend will be hosted separately from the backend services and will communicate with the backend through API calls.

### Backend

```text
Cloudflare Workers
TypeScript
REST APIs
```

Each backend service will be deployed as a separate Cloudflare Worker.

### Database

```text
Supabase PostgreSQL
```

Each microservice will use its own Supabase-hosted PostgreSQL database.

### Hosting and Infrastructure

```text
Cloudflare DNS
Cloudflare HTTPS
Cloudflare Workers
Supabase PostgreSQL
Supabase Storage
```

### Optional Tools

```text
pgAdmin
GitHub Actions
Wrangler CLI
```

Docker and Nginx are not required for the initial deployment because the backend services are deployed as serverless Cloudflare Workers.

---

## Planned Domain Structure

```text
plate.aareid.ca
```

Frontend Next.js PWA.

```text
api.plate.aareid.ca
```

Public API Gateway.

```text
auth-api.plate.aareid.ca
```

Auth and Tenant Service.

```text
fitness-api.plate.aareid.ca
```

Fitness Logging Service.

```text
social-api.plate.aareid.ca
```

Social Feed Service.

The frontend should only call the public API Gateway:

```text
api.plate.aareid.ca
```

The API Gateway will route requests to the correct backend service.

---

## High-Level Architecture

```text
User Browser / Mobile PWA
        |
        v
plate.aareid.ca
Next.js + TypeScript PWA
        |
        v
api.plate.aareid.ca
Cloudflare Worker API Gateway
        |
        v
+----------------------+------------------------+----------------------+
| Auth / Tenant Service| Fitness Logging Service| Social Feed Service  |
| Cloudflare Worker    | Cloudflare Worker      | Cloudflare Worker    |
+----------------------+------------------------+----------------------+
        |                       |                         |
        v                       v                         v
Supabase Auth DB       Supabase Fitness DB       Supabase Social DB
```

---

## Microservice Architecture

Plate is designed as a true microservice architecture.

Each service is:

- Separately deployed
- Accessed through API calls
- Responsible for one business domain
- Connected to its own database
- Independent from the frontend application

The frontend does not contain backend business logic. It communicates with the backend through the API Gateway.

---

## Folder Structure


```text
plate/
├── README.md
├── .gitignore
├── package.json
├── pnpm-workspace.yaml
├── turbo.json
├── .env.example
│
├── apps/
│   └── web/
│       ├── package.json
│       ├── next.config.ts
│       ├── tsconfig.json
│       ├── public/
│       │   ├── manifest.json
│       │   ├── icons/
│       │   └── screenshots/
│       │
│       └── src/
│           ├── app/
│           │   ├── layout.tsx
│           │   ├── page.tsx
│           │   ├── globals.css
│           │   │
│           │   ├── auth/
│           │   │   ├── login/
│           │   │   │   └── page.tsx
│           │   │   └── register/
│           │   │       └── page.tsx
│           │   │
│           │   ├── dashboard/
│           │   │   └── page.tsx
│           │   │
│           │   ├── feed/
│           │   │   └── page.tsx
│           │   │
│           │   ├── profile/
│           │   │   └── [userId]/
│           │   │       └── page.tsx
│           │   │
│           │   ├── workouts/
│           │   │   ├── page.tsx
│           │   │   └── new/
│           │   │       └── page.tsx
│           │   │
│           │   ├── food/
│           │   │   ├── page.tsx
│           │   │   └── new/
│           │   │       └── page.tsx
│           │   │
│           │   └── tenant/
│           │       └── settings/
│           │           └── page.tsx
│           │
│           ├── components/
│           │   ├── ui/
│           │   │   ├── Button.tsx
│           │   │   ├── Input.tsx
│           │   │   ├── Card.tsx
│           │   │   └── Modal.tsx
│           │   │
│           │   ├── layout/
│           │   │   ├── Navbar.tsx
│           │   │   ├── Sidebar.tsx
│           │   │   └── MobileNav.tsx
│           │   │
│           │   ├── feed/
│           │   │   ├── FeedList.tsx
│           │   │   ├── PostCard.tsx
│           │   │   ├── LikeButton.tsx
│           │   │   └── CommentList.tsx
│           │   │
│           │   ├── workouts/
│           │   │   ├── WorkoutForm.tsx
│           │   │   ├── ExerciseInput.tsx
│           │   │   └── WorkoutCard.tsx
│           │   │
│           │   ├── food/
│           │   │   ├── FoodLogForm.tsx
│           │   │   ├── FoodItemInput.tsx
│           │   │   └── FoodLogCard.tsx
│           │   │
│           │   └── profile/
│           │       ├── ProfileHeader.tsx
│           │       └── ProfileFeed.tsx
│           │
│           ├── features/
│           │   ├── auth/
│           │   │   ├── api.ts
│           │   │   ├── hooks.ts
│           │   │   └── types.ts
│           │   │
│           │   ├── fitness/
│           │   │   ├── api.ts
│           │   │   ├── hooks.ts
│           │   │   └── types.ts
│           │   │
│           │   ├── social/
│           │   │   ├── api.ts
│           │   │   ├── hooks.ts
│           │   │   └── types.ts
│           │   │
│           │   └── tenant/
│           │       ├── api.ts
│           │       ├── hooks.ts
│           │       └── types.ts
│           │
│           ├── lib/
│           │   ├── api-client.ts
│           │   ├── auth-token.ts
│           │   ├── constants.ts
│           │   └── utils.ts
│           │
│           ├── hooks/
│           │   ├── useAuth.ts
│           │   └── useDebounce.ts
│           │
│           └── types/
│               ├── api.ts
│               ├── post.ts
│               ├── user.ts
│               ├── workout.ts
│               └── food.ts
│
├── services/
│   ├── api-gateway/
│   │   ├── package.json
│   │   ├── wrangler.toml
│   │   ├── tsconfig.json
│   │   └── src/
│   │       ├── index.ts
│   │       ├── routes/
│   │       │   ├── auth.routes.ts
│   │       │   ├── fitness.routes.ts
│   │       │   └── social.routes.ts
│   │       ├── middleware/
│   │       │   ├── auth.middleware.ts
│   │       │   ├── cors.middleware.ts
│   │       │   └── error.middleware.ts
│   │       ├── clients/
│   │       │   ├── auth.client.ts
│   │       │   ├── fitness.client.ts
│   │       │   └── social.client.ts
│   │       └── utils/
│   │           └── response.ts
│   │
│   ├── auth-tenant-service/
│   │   ├── package.json
│   │   ├── wrangler.toml
│   │   ├── tsconfig.json
│   │   ├── migrations/
│   │   │   └── 001_init_auth_tenant.sql
│   │   └── src/
│   │       ├── index.ts
│   │       ├── routes/
│   │       │   ├── auth.routes.ts
│   │       │   ├── tenant.routes.ts
│   │       │   └── profile.routes.ts
│   │       ├── controllers/
│   │       │   ├── auth.controller.ts
│   │       │   ├── tenant.controller.ts
│   │       │   └── profile.controller.ts
│   │       ├── services/
│   │       │   ├── auth.service.ts
│   │       │   ├── tenant.service.ts
│   │       │   └── profile.service.ts
│   │       ├── repositories/
│   │       │   ├── user.repository.ts
│   │       │   ├── tenant.repository.ts
│   │       │   └── profile.repository.ts
│   │       ├── middleware/
│   │       │   ├── require-auth.ts
│   │       │   └── require-tenant-admin.ts
│   │       ├── db/
│   │       │   └── supabase.ts
│   │       ├── schemas/
│   │       │   ├── auth.schema.ts
│   │       │   ├── tenant.schema.ts
│   │       │   └── profile.schema.ts
│   │       └── types/
│   │           ├── user.ts
│   │           └── tenant.ts
│   │
│   ├── fitness-service/
│   │   ├── package.json
│   │   ├── wrangler.toml
│   │   ├── tsconfig.json
│   │   ├── migrations/
│   │   │   └── 001_init_fitness.sql
│   │   └── src/
│   │       ├── index.ts
│   │       ├── routes/
│   │       │   ├── workout.routes.ts
│   │       │   └── food.routes.ts
│   │       ├── controllers/
│   │       │   ├── workout.controller.ts
│   │       │   └── food.controller.ts
│   │       ├── services/
│   │       │   ├── workout.service.ts
│   │       │   └── food.service.ts
│   │       ├── repositories/
│   │       │   ├── workout.repository.ts
│   │       │   ├── exercise.repository.ts
│   │       │   ├── food-log.repository.ts
│   │       │   └── food-item.repository.ts
│   │       ├── clients/
│   │       │   └── auth.client.ts
│   │       ├── db/
│   │       │   └── supabase.ts
│   │       ├── schemas/
│   │       │   ├── workout.schema.ts
│   │       │   └── food.schema.ts
│   │       └── types/
│   │           ├── workout.ts
│   │           └── food.ts
│   │
│   └── social-feed-service/
│       ├── package.json
│       ├── wrangler.toml
│       ├── tsconfig.json
│       ├── migrations/
│       │   └── 001_init_social.sql
│       └── src/
│           ├── index.ts
│           ├── routes/
│           │   ├── post.routes.ts
│           │   ├── feed.routes.ts
│           │   ├── comment.routes.ts
│           │   ├── like.routes.ts
│           │   └── follow.routes.ts
│           ├── controllers/
│           │   ├── post.controller.ts
│           │   ├── feed.controller.ts
│           │   ├── comment.controller.ts
│           │   ├── like.controller.ts
│           │   └── follow.controller.ts
│           ├── services/
│           │   ├── post.service.ts
│           │   ├── feed.service.ts
│           │   ├── comment.service.ts
│           │   ├── like.service.ts
│           │   └── follow.service.ts
│           ├── repositories/
│           │   ├── post.repository.ts
│           │   ├── comment.repository.ts
│           │   ├── like.repository.ts
│           │   └── follow.repository.ts
│           ├── clients/
│           │   ├── auth.client.ts
│           │   └── fitness.client.ts
│           ├── db/
│           │   └── supabase.ts
│           ├── schemas/
│           │   ├── post.schema.ts
│           │   ├── comment.schema.ts
│           │   └── follow.schema.ts
│           └── types/
│               ├── post.ts
│               ├── comment.ts
│               └── follow.ts
│
├── packages/
│   ├── shared/
│   │   ├── package.json
│   │   └── src/
│   │       ├── types/
│   │       │   ├── user.ts
│   │       │   ├── tenant.ts
│   │       │   ├── post.ts
│   │       │   ├── workout.ts
│   │       │   └── food.ts
│   │       ├── constants/
│   │       │   ├── roles.ts
│   │       │   ├── post-types.ts
│   │       │   └── visibility.ts
│   │       └── utils/
│   │           ├── dates.ts
│   │           └── validation.ts
│   │
│   └── config/
│       ├── eslint/
│       └── typescript/
│
└── docs/
    ├── diagrams/
    │   ├── architecture.puml
    │   ├── deployment.puml
    │   ├── class-diagram.puml
    │   ├── erd.puml
    │   └── activity-diagrams/
    │       ├── create-account.puml
    │       ├── create-workout-post.puml
    │       └── configure-tenant.puml
    └── api/
        ├── auth-tenant-service.md
        ├── fitness-service.md
        └── social-feed-service.md
```


## Services

### 1. API Gateway

The API Gateway is the single public entry point for the frontend.

Responsibilities:

- Route requests to the correct service
- Validate authentication tokens where needed
- Hide internal service URLs from the frontend
- Keep frontend API calls simple and consistent

Example routes:

```text
/api/auth/*
/api/fitness/*
/api/social/*
```

Routing example:

```text
/api/auth/*     -> Auth / Tenant Service
/api/fitness/*  -> Fitness Logging Service
/api/social/*   -> Social Feed Service
```

---

### 2. Auth / Tenant Service

The Auth / Tenant Service manages users, authentication, tenants, roles, and tenant configuration.

Responsibilities:

- User registration
- User login
- Authentication
- Tenant creation
- Tenant settings
- User profiles
- Role management
- Tenant admin permissions

Main entities:

```text
tenants
tenant_settings
users
user_profiles
```

User roles:

```text
TENANT_ADMIN
USER
```

A tenant admin can do everything a regular user can do, plus configure the tenant account.

---

### 3. Fitness Logging Service

The Fitness Logging Service manages workout tracking and food logging.

Responsibilities:

- Create workouts
- Track exercises
- Track workout sets
- Log meals
- Log food items
- Store calories and macro information
- Store workout and food history

Main entities:

```text
workouts
exercises
workout_sets
food_logs
food_items
```

This service stores external references such as:

```text
user_id
tenant_id
post_id
```

These IDs refer to data owned by other services.

---

### 4. Social Feed Service

The Social Feed Service manages posts, likes, comments, follows, and user feeds.

Responsibilities:

- Create posts
- Display platform feed
- Display user profile feeds
- Like posts
- Comment on posts
- Follow users
- Store post visibility
- Connect social posts to workouts or food logs

Main entities:

```text
posts
comments
likes
follows
```

Post types:

```text
WORKOUT
FOOD
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

---

## Database Strategy

Each microservice owns its own database.

```text
Auth / Tenant Service  -> Supabase Auth DB
Fitness Logging Service -> Supabase Fitness DB
Social Feed Service -> Supabase Social DB
```

Services should not directly query each other’s databases.

Bad:

```text
Social Feed Service directly queries Fitness DB
```

Good:

```text
Social Feed Service calls Fitness Logging Service API
```

This keeps each service independent and prevents tight database coupling.

---

## Multi-Tenant Design

Plate is a multi-tenant platform.

A tenant represents a separate account, organization, community, or workspace inside the platform.

Each tenant has:

- Tenant name
- Tenant slug
- Tenant settings
- Tenant admin
- Users
- Posts
- Workouts
- Food logs

Users belong to a tenant through:

```text
tenant_id
```

Tenant admins can configure their own tenant account but do not control the entire platform.

---

## Main User Features

### Account Creation

Users can create an account and either:

- Create a new tenant account
- Join an existing tenant account

If a user creates a new tenant, they become the tenant admin.

### Workout Tracking

Users can create workout logs with:

- Workout title
- Workout type
- Duration
- Exercises
- Sets
- Reps
- Weight
- Notes

Workout logs can be shared as posts.

### Food Logging

Users can log meals with:

- Meal type
- Food items
- Calories
- Protein
- Carbohydrates
- Fat
- Notes

Food logs can also be shared as posts.

### Social Feed

Users can:

- View posts
- Like posts
- Comment on posts
- Follow users
- View user profiles
- View user profile feeds

### Tenant Configuration

Tenant admins can:

- Update tenant name
- Update tenant slug
- Configure default post visibility
- Enable or disable public discovery
- Configure comment settings
- Manage tenant profile settings

---

## Example API Endpoints

### Auth / Tenant Routes

```text
POST /api/auth/register
POST /api/auth/login
GET  /api/auth/me
PATCH /api/auth/profile
GET  /api/auth/tenant
PATCH /api/auth/tenant/settings
```

### Fitness Routes

```text
POST /api/fitness/workouts
GET  /api/fitness/workouts/:id
GET  /api/fitness/users/:userId/workouts

POST /api/fitness/food-logs
GET  /api/fitness/food-logs/:id
GET  /api/fitness/users/:userId/food-logs
```

### Social Routes

```text
POST /api/social/posts
GET  /api/social/feed
GET  /api/social/users/:userId/feed

POST /api/social/posts/:postId/like
DELETE /api/social/posts/:postId/like

POST /api/social/posts/:postId/comments
GET  /api/social/posts/:postId/comments

POST /api/social/users/:userId/follow
DELETE /api/social/users/:userId/follow
```

---

## Core Workflows

### User Creates Account

```text
User opens app
-> Selects create account
-> Enters account details
-> Creates or joins tenant
-> Auth / Tenant Service validates data
-> User record is created
-> User profile is created
-> Token is generated
-> User is redirected to dashboard
```

### User Creates Workout Post

```text
User enters workout details
-> Frontend validates form
-> Fitness Logging Service saves workout
-> Social Feed Service creates post
-> Post appears on user profile feed
-> Followers can view, like, and comment
```

### Tenant Admin Configures Tenant Account

```text
Tenant admin opens settings
-> Auth / Tenant Service validates role
-> Existing settings are displayed
-> Tenant admin updates settings
-> Auth / Tenant Service saves changes
-> Updated tenant settings are returned
```

---

## Planned Data Model

### Auth / Tenant Service

```text
tenants
- id
- name
- slug
- owner_user_id
- plan
- created_at

tenant_settings
- id
- tenant_id
- default_post_visibility
- allow_public_discovery
- allow_comments
- updated_at

users
- id
- tenant_id
- username
- email
- password_hash
- role
- created_at

user_profiles
- id
- user_id
- display_name
- bio
- profile_image_url
- fitness_goal
- updated_at
```

### Fitness Logging Service

```text
workouts
- id
- tenant_id
- user_id
- post_id
- title
- workout_type
- duration_minutes
- notes
- created_at

exercises
- id
- workout_id
- name
- muscle_group
- notes

workout_sets
- id
- exercise_id
- set_number
- reps
- weight
- rest_seconds

food_logs
- id
- tenant_id
- user_id
- post_id
- meal_type
- title
- calories
- protein
- carbs
- fat
- notes
- created_at

food_items
- id
- food_log_id
- name
- calories
- protein
- carbs
- fat
- quantity
- unit
```

### Social Feed Service

```text
posts
- id
- tenant_id
- user_id
- post_type
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

likes
- id
- post_id
- user_id
- created_at

follows
- id
- follower_user_id
- following_user_id
- created_at
```

---

## Development Approach

The project will be developed in phases.

### Phase 1: Foundation

- Set up Next.js TypeScript PWA
- Set up Cloudflare deployment
- Set up Supabase projects/databases
- Create initial database schemas
- Create API Gateway Worker

### Phase 2: Authentication and Tenants

- Register users
- Log in users
- Create tenant accounts
- Assign tenant admin role
- Create user profiles
- Add tenant settings

### Phase 3: Fitness Logging

- Create workout logs
- Add exercises
- Add workout sets
- Create food logs
- Add food items

### Phase 4: Social Feed

- Create posts
- Display feed
- Display user profile feed
- Add likes
- Add comments
- Add follows

### Phase 5: PWA Polish

- Add web app manifest
- Add service worker
- Improve mobile responsiveness
- Add install support
- Improve offline asset handling

### Phase 6: Deployment and CI/CD

- Deploy frontend
- Deploy API Gateway
- Deploy microservices
- Connect custom domains
- Add GitHub Actions
- Add basic automated checks

---

## Deployment Strategy

The project will avoid Docker, Nginx, and Kubernetes for the initial deployment.

Instead, it will use:

```text
Cloudflare Workers for APIs
Cloudflare DNS and HTTPS
Cloudflare-hosted frontend or compatible Next.js hosting
Supabase-hosted PostgreSQL databases
Supabase Storage for uploaded media
```

This keeps hosting costs low while still allowing a real microservice architecture.

---

## Future Improvements

Possible future features:

- Push notifications
- Workout streaks
- Food and workout analytics
- Progress charts
- Media uploads
- User search
- Tenant search
- Private accounts
- Content moderation
- Challenge system
- Leaderboards
- AI-generated workout summaries
- Barcode scanning for food items
- Wearable integration

---

## Architecture Summary

Plate is designed as a multi-tenant, microservice-based social fitness PWA.

The architecture uses:

```text
Next.js + TypeScript PWA
Cloudflare Worker API Gateway
Cloudflare Worker microservices
Supabase PostgreSQL databases
Supabase Storage
```

The system is separated into three main business services:

```text
Auth / Tenant Service
Fitness Logging Service
Social Feed Service
```

This design keeps the frontend clean, separates backend responsibilities, supports future scaling, and avoids unnecessary infrastructure costs during the MVP stage.
