# Product Requirements Document (PRD)
# URL Shortener Platform – Production Scale

**Version:** 1.0  
**Status:** Draft  
**Product Type:** SaaS / Distributed Web Application  
**Document Type:** Technical Product Requirements  
**Target:** Production-grade, horizontally scalable platform

---

## 1. Product Overview

### 1.1 Product Name
Shortly – Scalable URL Management Platform

### 1.2 Purpose

Build a highly available, distributed URL shortening platform that converts long URLs into short, shareable links.

The platform should support:

- URL shortening and redirection.
- Custom aliases and branded domains.
- Link expiration and lifecycle management.
- Detailed click analytics.
- QR code generation.
- User authentication and authorization.
- API-based URL creation.
- Rate limiting and abuse prevention.
- High-throughput redirection.
- Enterprise account management.

### 1.3 Vision

Provide a reliable URL management infrastructure capable of serving millions of users and billions of redirects while maintaining low latency, high availability, and accurate analytics.

---

## 2. Goals and Objectives

### Primary Goals

1. Generate unique short URLs.
2. Redirect users to original URLs with minimal latency.
3. Support millions of URL records.
4. Handle high-volume concurrent redirects.
5. Provide real-time and historical analytics.
6. Support custom domains and aliases.
7. Provide secure public and authenticated APIs.
8. Support horizontal scaling.

### Non-Goals (Initial Release)

- Full marketing automation.
- Email campaign management.
- Social media scheduling.
- Browser extensions.
- Native mobile applications.
- Advanced A/B testing.

---

## 3. User Personas

### Persona 1: Guest User
- Shortens URLs without registration.
- Copies and shares generated links.
- Has limited usage quotas.

### Persona 2: Registered User
- Creates and manages short URLs.
- Views link analytics.
- Updates destination URLs.
- Creates custom aliases.
- Manages expiration dates.

### Persona 3: Business User
- Uses branded domains.
- Generates links through APIs.
- Accesses detailed analytics.
- Manages team members.
- Uses bulk URL creation.

### Persona 4: Administrator
- Monitors system health.
- Manages users.
- Detects malicious URLs.
- Handles abuse reports.
- Manages platform-wide configurations.

---

## 4. Functional Requirements

### 4.1 Authentication and Authorization

**Features:**
- User registration.
- Email verification.
- Login and logout.
- Password reset.
- Google OAuth (optional).
- Access token and refresh token management.
- Role-based access control.

**Roles:**
- Guest
- User
- Team Admin
- Platform Admin

**Requirements:**
- Secure password hashing.
- JWT-based authentication or equivalent session management.
- Refresh token rotation.
- Account suspension.
- Login attempt throttling.

### 4.2 URL Shortening

Users should be able to submit a long URL and receive a shortened URL.

**Input:**
- Original URL.
- Optional custom alias.
- Optional expiration date.
- Optional title.
- Optional tags.

**Output:**
- Short URL.
- Short code.
- Creation timestamp.
- Expiration timestamp.

**Example:**

Original URL:

https://example.com/products/electronics/laptops/macbook-pro-2026

Short URL:

https://sho.rt/aB92xK

**Business Rules:**
- Short codes must be unique.
- URLs must use permitted HTTP/HTTPS schemes.
- Invalid URLs must be rejected.
- Custom aliases must be unique within their domain.
- Expired links must not redirect.
- Reserved aliases must be blocked.
- Deleted aliases should have a defined reuse policy.

### 4.3 Short Code Generation

The system should support a scalable short-code generation strategy.

**Recommended approach:**

- Generate a unique numeric identifier.
- Encode the identifier using Base62.
- Use the encoded value as the short code.

Base62 characters:

`abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789`

Example:

| Numeric ID | Base62 Code |
|---|---|
| 1000 | g8 |
| 100000 | q0U |
| 1000000 | 4c92 |

**Requirements:**
- No duplicate short codes.
- Support distributed ID generation.
- Avoid predictable sequential public identifiers where necessary.
- Support future migration to a different encoding strategy.

### 4.4 URL Redirection

When a user accesses a shortened URL:

1. Extract the short code from the request.
2. Identify the domain.
3. Check Redis cache.
4. If unavailable, query the primary data store.
5. Check expiration and link status.
6. Return the appropriate redirect response.
7. Publish click information asynchronously.

**Redirect response:**
- HTTP 302 by default.
- HTTP 301 optionally configurable.
- HTTP 404 for unknown links.
- HTTP 410 for expired or permanently removed links.
- HTTP 403 for blocked links.

**Performance target:**
- P95 redirect latency below 50 ms under normal production conditions, excluding client network latency.

### 4.5 Custom Aliases

Users can specify their own short codes.

Example:

`https://sho.rt/myportfolio`

Requirements:
- Alias uniqueness.
- Character validation.
- Reserved keyword protection.
- Case-sensitivity policy.
- Ownership validation.
- Alias update restrictions.

### 4.6 Link Management

Authenticated users should be able to:

- View all created links.
- Search links.
- Filter by creation date.
- Filter by active, expired, and disabled status.
- Update destination URLs.
- Edit titles and descriptions.
- Delete links.
- Enable or disable links.
- Set expiration dates.
- Add tags.

### 4.7 QR Code Generation

Generate downloadable QR codes for shortened URLs.

Supported formats:
- PNG
- SVG

Requirements:
- Configurable QR dimensions.
- Optional foreground and background colors.
- QR codes must encode the final short URL.

### 4.8 Click Analytics

The platform should provide detailed link analytics.

**Metrics:**

- Total clicks.
- Unique visitors (estimated).
- Clicks over time.
- Country distribution.
- City distribution where available.
- Device types.
- Browser types.
- Operating systems.
- Referrer domains.
- Top-performing links.

**Analytics requirements:**
- Asynchronous event processing.
- Historical reporting.
- Time-based aggregation.
- Bot and crawler filtering.
- Privacy-conscious IP processing.
- Duplicate event handling.

### 4.9 API Management

Provide REST APIs for external integrations.

Features:
- API key generation.
- API key revocation.
- API usage tracking.
- Rate limits.
- API documentation.
- Request authentication.

### 4.10 Branded Domains

Business customers can configure their own domains.

Example:

`https://go.company.com/abc123`

Requirements:
- Domain ownership verification.
- DNS configuration verification.
- TLS certificate provisioning.
- Domain-to-account mapping.
- Custom domain routing.
- Domain suspension and removal.

---

## 5. Non-Functional Requirements

| Requirement | Target |
|---|---|
| Availability | 99.95% monthly |
| Redirect latency | P95 < 50 ms |
| API latency | P95 < 200 ms |
| Initial capacity | 1,000 redirect requests/sec |
| Horizontal scalability | Required |
| Data durability | 99.999999% design objective |
| Authentication | Secure token-based authentication |
| Encryption | TLS in transit, encryption at rest |
| Monitoring | Metrics, logs, traces |
| Disaster recovery | Defined RPO/RTO |
| Data retention | Configurable policies |

These are initial engineering targets and must be validated through load testing.

### Scalability Requirements

- Stateless API instances.
- Horizontal scaling.
- Redis caching.
- Asynchronous event processing.
- Database indexing.
- Read replicas where appropriate.
- Database partitioning when necessary.
- CDN and edge caching where redirect semantics permit.

---

## 6. System Architecture

### 6.1 High-Level Architecture

Client Applications:
- Web Dashboard
- External API Clients
- Browser / Mobile Clients

Infrastructure:

1. Load Balancer
2. API Gateway
3. Authentication Service
4. URL Management Service
5. Redirect Service
6. Analytics Service
7. Background Workers
8. Notification Service

Infrastructure Components:
- PostgreSQL
- Redis
- Kafka / RabbitMQ
- ClickHouse
- Object Storage
- Prometheus
- Grafana

### 6.2 Service Responsibilities

**API Gateway**
- Request routing.
- Rate limiting.
- Authentication enforcement.
- Request validation.

**Authentication Service**
- User registration.
- Login.
- Token management.
- Role management.

**URL Service**
- URL creation.
- Short-code generation.
- Link management.
- Domain management.

**Redirect Service**
- Short-code resolution.
- Cache lookup.
- Destination retrieval.
- Redirect response.
- Click event publishing.

**Analytics Service**
- Event consumption.
- Event deduplication.
- Aggregation.
- Analytics queries.

**Background Workers**
- Expired URL processing.
- Abuse detection.
- Scheduled cleanup.
- Analytics aggregation.

### 6.3 Redirect Architecture

Redirects are the highest-priority operation because every shared short URL depends on them.

Request flow:

1. Client requests `sho.rt/aB92xK`.
2. Load balancer forwards request.
3. Redirect service extracts domain and code.
4. Redis cache is queried.
5. Cache hit returns the destination.
6. Cache miss queries PostgreSQL.
7. Result is cached.
8. Redirect response is returned.
9. Click event is published asynchronously.

The redirect should not wait for analytics processing.

---

## 7. Database Design

### 7.1 Users Table

| Column | Type | Description |
|---|---|---|
| id | UUID | Primary key |
| name | VARCHAR | User name |
| email | VARCHAR | Unique email |
| password_hash | TEXT | Password hash |
| role | VARCHAR | User role |
| status | VARCHAR | Account status |
| created_at | TIMESTAMP | Creation time |
| updated_at | TIMESTAMP | Update time |

### 7.2 URLs Table

| Column | Type | Description |
|---|---|---|
| id | BIGINT | Primary key |
| user_id | UUID | Owner |
| domain_id | UUID | Custom domain |
| short_code | VARCHAR | Short identifier |
| original_url | TEXT | Destination |
| title | VARCHAR | Link title |
| status | VARCHAR | Active/Disabled |
| expires_at | TIMESTAMP | Expiration |
| created_at | TIMESTAMP | Creation time |
| updated_at | TIMESTAMP | Update time |

**Constraints:**
- Unique(domain_id, short_code).
- Index(user_id).
- Index(created_at).
- Index(expires_at).
- Index(status).

### 7.3 Domains Table

| Column | Type | Description |
|---|---|---|
| id | UUID | Primary key |
| user_id | UUID | Owner |
| domain_name | VARCHAR | Custom domain |
| verification_status | VARCHAR | Verification |
| ssl_status | VARCHAR | TLS status |
| created_at | TIMESTAMP | Creation time |

### 7.4 Click Events Table

| Column | Type | Description |
|---|---|---|
| id | UUID | Event ID |
| url_id | BIGINT | Associated URL |
| timestamp | TIMESTAMP | Click time |
| country | VARCHAR | Country |
| city | VARCHAR | City |
| device_type | VARCHAR | Device |
| browser | VARCHAR | Browser |
| referrer | TEXT | Referrer |
| visitor_hash | VARCHAR | Privacy-preserving visitor identifier |
| is_bot | BOOLEAN | Bot classification |

For production, high-volume click events should be stored in an analytical database rather than relying on a heavily written transactional PostgreSQL table.

### 7.5 API Keys Table

| Column | Type | Description |
|---|---|---|
| id | UUID | Primary key |
| user_id | UUID | Owner |
| key_hash | TEXT | Hashed API key |
| name | VARCHAR | Key name |
| status | VARCHAR | Active/Revoked |
| last_used_at | TIMESTAMP | Last usage |
| created_at | TIMESTAMP | Creation time |

---

## 8. API Specifications

Base URL:

`https://api.sho.rt/v1`

### Authentication

| Method | Endpoint | Description |
|---|---|---|
| POST | /auth/register | Register |
| POST | /auth/login | Login |
| POST | /auth/refresh | Refresh token |
| POST | /auth/logout | Logout |
| POST | /auth/forgot-password | Reset password |

### URL Management

| Method | Endpoint | Description |
|---|---|---|
| POST | /urls | Create short URL |
| GET | /urls | List URLs |
| GET | /urls/{id} | Get URL |
| PATCH | /urls/{id} | Update URL |
| DELETE | /urls/{id} | Delete URL |
| POST | /urls/{id}/disable | Disable URL |
| POST | /urls/{id}/enable | Enable URL |
| GET | /urls/{id}/qr | Generate QR code |

### Redirection

| Method | Endpoint | Description |
|---|---|---|
| GET | /{short_code} | Redirect |
| HEAD | /{short_code} | Resolve redirect metadata |

Custom-domain requests should resolve using the incoming host and short code.

### Analytics

| Method | Endpoint | Description |
|---|---|---|
| GET | /analytics/overview | Account overview |
| GET | /analytics/urls/{id} | Link analytics |
| GET | /analytics/urls/{id}/countries | Country statistics |
| GET | /analytics/urls/{id}/devices | Device statistics |
| GET | /analytics/urls/{id}/timeline | Click timeline |

### API Keys

| Method | Endpoint | Description |
|---|---|---|
| POST | /api-keys | Create API key |
| GET | /api-keys | List API keys |
| DELETE | /api-keys/{id} | Revoke API key |

---

## 9. API Request and Response Examples

### Create Short URL

**POST** `/v1/urls`

Request:

```json
{
  "original_url": "https://example.com/products/123",
  "custom_alias": "product123",
  "expires_at": null,
  "title": "Product Page"
}
```

Response:

```json
{
  "id": "uuid",
  "short_code": "product123",
  "short_url": "https://sho.rt/product123",
  "original_url": "https://example.com/products/123",
  "status": "active",
  "created_at": "2026-11-03T10:00:00Z"
}
```

### Redirect

**GET** `/product123`

Response:

```http
HTTP/1.1 302 Found
Location: https://example.com/products/123
Cache-Control: no-store
```

---

## 10. Caching Strategy

Redis should be the primary cache for frequently accessed short URLs.

**Cache key:**

`url:{domain_id}:{short_code}`

**Cache value:**

```json
{
  "url_id": 12345,
  "original_url": "https://example.com",
  "status": "active",
  "expires_at": null
}
```

**Caching policies:**
- Cache frequently accessed URLs.
- Use TTL-based expiration.
- Invalidate cache after destination updates.
- Invalidate cache after link deletion or suspension.
- Protect against cache stampedes.
- Apply negative caching for frequently requested invalid codes.
- Avoid caching sensitive or rapidly changing link