# Airbnb Clone – Backend

A robust, scalable backend for an Airbnb-style marketplace that manages users, property listings, bookings, payments, and reviews. Built for correctness, performance, and developer velocity.

---

## 📑 Contents

* [🚀 Objective](#-objective)
* [🏆 Project Goals](#-project-goals)
* [🛠️ Features Overview](#️-features-overview)
* [⚙️ Technology Stack](#️-technology-stack)
* [👥 Team Roles](#-team-roles)
* [📈 API Documentation Overview](#-api-documentation-overview)
* [🗃️ Database Design](#️-database-design)
* [🔩 Feature Breakdown](#-feature-breakdown)
* [🔐 API Security](#-api-security)
* [🚢 CI/CD Pipeline](#-cicd-pipeline)
* [📦 Project Initialization](#-project-initialization-mandatory)
* [📚 Technology Stack (Details)](#-technology-stack-details)
* [🧭 Development Conventions](#-development-conventions)
* [✅ Next Steps](#-next-steps)

---

## 🚀 Objective

Provide a solid foundation for user interactions, property listings, bookings, and payments, exposing clean REST and GraphQL APIs and supporting operational needs like observability, security, and CI/CD.

---

## 🏆 Project Goals

* **User Management**: Secure registration, authentication, and profile management.
* **Property Management**: Create, update, search, and retrieve property listings.
* **Booking System**: Reserve properties with availability checks and lifecycle management.
* **Payment Processing**: Process and record transactions safely and idempotently.
* **Review System**: Post moderated reviews and ratings.
* **Data Optimization**: Fast reads via indexing, caching, and pagination.

---

## 🛠️ Features Overview

### 1) API Documentation

* **OpenAPI**: REST endpoints documented for easy client consumption.
* **Django REST Framework (DRF)**: Batteries‑included REST CRUD.
* **GraphQL**: Flexible data fetching for web and mobile clients.

### 2) User Authentication

* **Endpoints**: `/users/`, `/users/{user_id}/`
* **Features**: Sign‑up, login, JWT issuance/refresh, profile CRUD.

### 3) Property Management

* **Endpoints**: `/properties/`, `/properties/{property_id}/`
* **Features**: CRUD, search, filtering (price, location, amenities), availability.

### 4) Booking System

* **Endpoints**: `/bookings/`, `/bookings/{booking_id}/`
* **Features**: Create/modify/cancel bookings; check‑in/out; prevent overlaps.

### 5) Payment Processing

* **Endpoints**: `/payments/`
* **Features**: Capture/confirm/refund; webhook handling; idempotency.

### 6) Review System

* **Endpoints**: `/reviews/`, `/reviews/{review_id}/`
* **Features**: Post/edit/delete reviews; rating aggregation; moderation flags.

### 7) Database Optimizations

* **Indexing**: Hot paths (users.email, properties.host\_id, bookings.date ranges).
* **Caching**: Redis for hot listings, search results, and derived aggregates.

---

## ⚙️ Technology Stack

* **Django**: High‑level web framework and ORM.
* **Django REST Framework**: REST API layer (auth, pagination, throttling, serializers).
* **PostgreSQL**: Primary relational datastore with advanced features (indices, constraints, JSONB, GIST/GIN).
* **GraphQL (Graphene or Strawberry)**: Flexible data access; avoids over/under‑fetching.
* **Celery**: Async background jobs (emails, webhooks, payment reconciliation).
* **Redis**: Task broker & cache (rate limiting, sessions, query caching).
* **Docker**: Dev/prod parity via containers.
* **CI/CD Pipelines**: GitHub Actions for lint/test/build/migrate/deploy.

---

## 👥 Team Roles

> Inspired by the project overview and standard SDLC role definitions.

* **Backend Developer**: Designs domain models; implements REST/GraphQL endpoints, business logic, validations; writes tests; handles performance profiling.
* **Database Administrator (DBA)**: Owns schema evolution, indexing strategy, backups, PITR, and performance tuning.
* **DevOps Engineer**: Builds CI/CD, container orchestration, observability (logs/metrics/traces), scaling, and secret management.
* **QA Engineer**: Test plans, automated API/UI tests, regression suites, load tests, and release sign‑off.
* **Security Engineer** *(optional/overlapping)*: Threat modeling, dependency scanning, secret rotation, incident response.
* **Product Manager** *(optional)*: Requirements, prioritization, roadmap, and acceptance criteria.

---

## 📈 API Documentation Overview

* **REST (OpenAPI/Swagger)**: `/api/docs` with schemas for Users, Properties, Bookings, Payments, Reviews.
* **GraphQL**: `/graphql` endpoint with schema & interactive playground enabled in non‑prod.

### REST Endpoint Summary

**Users**

* `GET /users/` – List users
* `POST /users/` – Create user
* `GET /users/{user_id}/` – Retrieve user
* `PUT /users/{user_id}/` – Update user
* `DELETE /users/{user_id}/` – Delete user

**Properties**

* `GET /properties/` – List properties
* `POST /properties/` – Create property
* `GET /properties/{property_id}/` – Retrieve property
* `PUT /properties/{property_id}/` – Update property
* `DELETE /properties/{property_id}/` – Delete property

**Bookings**

* `GET /bookings/` – List bookings
* `POST /bookings/` – Create booking
* `GET /bookings/{booking_id}/` – Retrieve booking
* `PUT /bookings/{booking_id}/` – Update booking
* `DELETE /bookings/{booking_id}/` – Delete booking

**Payments**

* `POST /payments/` – Process payment

**Reviews**

* `GET /reviews/` – List reviews
* `POST /reviews/` – Create review
* `GET /reviews/{review_id}/` – Retrieve review
* `PUT /reviews/{review_id}/` – Update review
* `DELETE /reviews/{review_id}/` – Delete review

---

## 🗃️ Database Design

**Core Entities & Key Fields**

**User**

* `id` (UUID, PK)
* `email` (unique), `password_hash`
* `first_name`, `last_name`, `phone`
* `role` (guest | host | admin)
* `is_verified`, `created_at`, `updated_at`

**Property**

* `id` (UUID, PK), `host_id` (FK → User)
* `title`, `description`, `type` (entire | private\_room | shared\_room)
* `address`, `city`, `country`
* `location` (lat, lon) or PostGIS point
* `amenities` (JSONB), `photos` (array/JSONB)
* `price_per_night`, `currency`, `max_guests`, `bedrooms`, `bathrooms`
* `status` (active | paused | drafted)

**Booking**

* `id` (UUID, PK), `property_id` (FK), `guest_id` (FK → User)
* `start_date`, `end_date`, `guests_count`
* `total_amount`, `currency`, `status` (pending | confirmed | cancelled | completed)
* `created_at`

**Payment**

* `id` (UUID, PK), `booking_id` (FK)
* `provider` (e.g., Stripe/Paystack), `amount`, `currency`
* `status` (requires\_action | succeeded | failed | refunded)
* `transaction_ref`, `paid_at`, `metadata` (JSONB)

**Review**

* `id` (UUID, PK), `property_id` (FK), `author_id` (FK → User), `booking_id` (FK)
* `rating` (1–5), `comment`, `is_public`, `created_at`

**Relationships**

* A **User(host)** `1‑N` **Property**
* A **Property** `1‑N` **Booking**
* A **User(guest)** `1‑N` \*\*Booking\`
* A **Booking** `1‑1` **Payment** *(or 1‑N with retries/refunds)*
* A **Property** `1‑N` **Review**; **Review** references **Booking** for verification

**Indexes & Constraints**

* `users(email)` unique index
* `properties(host_id)`, `properties(status)`
* **Geo**: GIST on `properties(location)` for proximity search
* **Search**: GIN on `amenities`, trigram on `title`/`city` (optional)
* **Dates**: composite index `(property_id, start_date, end_date)`
* **No overlapping bookings**: PostgreSQL exclusion constraint on date ranges per `property_id`

---

## 🔩 Feature Breakdown

* **User Management**: Sign‑up/login with JWT (access/refresh), email verification, password reset, profile edit, role‑based views (guest/host/admin).
* **Property Management**: Hosts create and manage listings, upload photos, set pricing and availability, and toggle status; clients browse with rich filters.
* **Booking System**: Real‑time availability checks, conflict prevention, cancellation policies, and host/guest notifications.
* **Payment Processing**: Initiate and confirm charges, handle 3‑DS/OTP flows where applicable, verify webhooks, support refunds, and ensure idempotency.
* **Reviews**: Guests post ratings/comments tied to completed stays; moderation and spam controls.
* **Admin Ops**: Dashboards for user/property moderation, dispute handling, refunds, and metrics.

---