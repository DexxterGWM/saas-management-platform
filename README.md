# SaaS Management Platform

SaaS Management Platform developed for managing organizations, users, and business operations in multi-tenant environments, providing access control, asynchronous processing, and integration with external services.

<br/>

> [!NOTE]
> MVP is currently under development.

<br/><br/>



---

## Overview

This Platform is a production-oriented full-stack application designed to demonstrate modern backend architecture, scalable application design, and enterprise development practices.

It allows organizations to manage isolated workspaces, users, projects, subscriptions, and business operations while integrating authentication, background processing, payment workflows, and external services into a unified architecture.

<br/><br/>



---

## Architecture

The application follows a modular architecture where each domain encapsulates its own business logic while sharing common infrastructure and application services.

<br/>

### System diagram:
```mermaid
flowchart TD

Client[Client - Next.js / React] -->|HTTPS| API[NestJS API Layer]

API --> Auth[Auth Module]
API --> SaaS[SaaS Core Module]
API --> Payments[Payments Module]

%% DB Separation (logical, even if same instance)
Auth --> AuthDB[(PostgreSQL - Auth)]
SaaS --> SaaSDB[(PostgreSQL - Core)]
Payments --> PayDB[(PostgreSQL - Payments)]

%% Redis roles EXPLICIT
API --> Cache[(Redis - Cache / Sessions)]
API --> Queue[(Redis - BullMQ Queue)]

Queue --> Worker["Worker (BullMQ Processor)"]

%% Payments integration
SaaS -->|send email / track events| External["External APIs (Email - SendGrid, CRM - HubSpot, Analytics - Segment)"]

Payments --> Stripe[Stripe API]
Stripe --> |WEBHOOK| Payments
```

---

### Request Flow diagram:
```mermaid
sequenceDiagram

actor User

participant Client as Next.js Client
participant API as NestJS API
participant Auth
participant SaaS
participant Payments
participant SaaSDB
participant PayDB
participant Cache as Redis (Cache)
participant Queue as BullMQ Queue
participant Worker
participant Stripe

User ->> Client: Action
Client ->> API: HTTP Request (JWT)

API ->> Auth: Validate Token
Auth ->> Cache: Check Session
Auth -->> API: ok
Auth -->> API: unauthorized (invalid or expired token)

API ->> SaaS: Execute business logic
SaaS ->> SaaSDB: SELECT projects WHERE org_id
SaaSDB -->> SaaS: rows

SaaS ->> SaaSDB: INSERT Project
SaaSDB -->> SaaS: project_id

SaaS ->> Queue: Enqueue Job
Queue -> Worker: Process Job
Worker ->> SaaSDB: Update Job Result
SaaSDB -->> Worker: ok
Worker -->> Queue: failed (retry with backoff)

SaaS -->> API: ProjectListDTO
API -->> Client: ProjectListResponse

Note over Payments, Stripe: Payment Flow
Client ->> API: Create subscription
API ->> Payments: Create subscription (idempotency_key)
Payments ->> PayDB: INSERT subscription (idempotency_key)
PayDB -->> Payments: subscription (existing or null)

alt subscription exists
	Payments -->> API: existing subscription
else new
	Payments ->> Stripe: Create subscription
	Stripe -->> Payments: subscription_id
end

API -->> Client: checkout_url (stripe hosted)
Client ->> Stripe: Open checkout page

Stripe ->> Payments: Webhook Event
Payments ->> PayDB: UPDATE subscription SET status=active
Payments ->> Queue: Emit subscription.activated
Queue -> Worker: Send email / update CRM
PayDB -->> Payments: ok

Payments -->> Stripe: 400 (invalid signature)
```

<br/><br/>



---

## Features

### Authentication:
- JWT authentication
- OAuth integration (Google and GitHub)
- Refresh tokens
- Session management

### Authorization:
- Role-Based Access Control (RBAC)
- Organization-level permissions
- Workspace isolation

### Business Logic:
- Multi-tenant architecture
- Subscription management
- Payment workflows
- External API integrations

### Performance:
- Cursor pagination
- Optimized database queries
- Redis caching
- Background job processing

### Infrastructure:
- Dockerized environment
- CI/CD pipelines
- Automated testing
- OpenAPI documentation

<br/><br/>



---

## Technology Stack

<div align="center">

| Category | Technologies |
|-----------|--------------|
| Frontend | React, Next.js |
| Backend | Node.js, NestJS |
| Database | PostgreSQL |
| ORM | Drizzle ORM |
| Cache | Redis |
| Queue | BullMQ |
| Authentication | JWT, OAuth |
| Payments | Stripe |
| Documentation | OpenAPI / Swagger |
| Testing | Jest, Playwright |
| Infrastructure | Docker, GitHub Actions |
| Deployment | Railway, Vercel |

</div>

<br/><br/>



---

## Getting Started

<br/>

> [!NOTE]
> Make sure to configure the `.env` file using the values provided in `.env.example`.

### Prerequisites:
- Node.js
- Docker
- PostgreSQL
- Redis

<br/><br/>



---

## Installation:
```bash
git clone https://github.com/DexxterGWM/saas-management-platform.git
cd saas-management-platform
npm install
```

<br/><br/>



---

## Repository
[saas-management-platform](https://github.com/DexxterGWM/saas-management-platform) - Made by: [@DexxterGWM](https://github.com/DexxterGWM)

<br/><br/>



---

## License

This project is intended for educational purposes and portfolio demonstration.

Built to explore production-ready software architecture, scalable backend development, and modern SaaS engineering practices.
