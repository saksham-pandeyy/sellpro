# Cloud Infrastructure

The application is deployed on a cloud provider using a cost-optimized, scalable architecture. Static assets are served through a CDN, the backend runs on compute instances, and a managed database handles persistent storage. This document covers the infrastructure design, networking, security, monitoring, cost optimization, and the reasoning behind each architectural choice.

---

## Architecture Overview

```
DNS
    |
CDN ----> Storage (Origin Fallback)
    |
    +--> Load Balancer
              |
      +-------+-------+
      |               |
    App Server     App Server
      |               |
      +-------+-------+
              |
    +---------+---------+
    |         |         |
  Database    Cache    Storage
```

### Traffic Flow

1. **DNS resolution** routes traffic to the CDN for the frontend and to the load balancer for API requests
2. **Static content** (JS bundles, CSS, images) is served entirely from CDN edge locations. Users download the app from the nearest edge location
3. **API requests** go through the CDN to the load balancer, which distributes traffic across application servers in different availability zones
4. **Database queries** hit replicas for reporting workloads and the primary for write operations
5. **Background jobs** run on separate processes, consuming from a queue

---

## Component Architecture

### DNS

DNS routing with latency-based policies:
- The main domain resolves to the CDN for frontend content
- The API subdomain resolves to the load balancer
- Latency-based routing directs users to the nearest healthy endpoint
- Automatic failover if the primary region becomes unhealthy

### CDN

The CDN serves the static React build:

| Setting | Value | Rationale |
| --------------------- | ---------------------------------- | ------------------------------------------- |
| Origin | Application server | Single origin for both app and API |
| Cache behavior (static) | Cache hashed asset files | Long cache duration (content-addressed) |
| Cache behavior (API) | No caching | Passes through to load balancer |
| Default TTL | Configurable | Balance between freshness and performance |
| Error pages | Custom redirect to index.html | SPA routing needs fallback |
| SSL certificate | Managed certificate provider | Free, auto-renewed |

**Why CDN for a static app?** The React app is fully client-side rendered with no server-side rendering. The CDN distributes JS bundles to edge locations worldwide. Users download from the nearest location. Zero server load for page delivery.

**Cache invalidation strategy:** Each build produces uniquely named files with content hashes. The CDN sees new file names on each deployment, so no cache invalidation is needed. Old files naturally expire when users stop requesting them.

### Load Balancer

The load balancer distributes traffic across application servers in multiple availability zones:
- **Target groups**: Separate groups for application servers and background workers
- **Health checks**: Endpoint expected to return 200
- **Sticky sessions**: Disabled (stateless design)
- **SSL termination**: At the load balancer
- **Access logs**: Logged to storage for audit and analysis

### Application Servers

Application servers run the backend services. The frontend is served by a lightweight static file server:

| Aspect | Configuration | Rationale |
| -------------------- | -------------------------------- | ------------------------------------------ |
| Instance type | Burstable, cost-effective | Handles variable load efficiently |
| Auto-scaling | Based on CPU/request metrics | Handles traffic spikes |
| Security | Inbound restricted to load balancer only | No direct internet access to instances |

**Why not containers?** The application is a single-process API with no complex orchestration needs. Traditional compute instances with automated deployment are simpler to manage for a team without dedicated DevOps resources. If the service grew to many microservices, container orchestration would be reconsidered.

### Database

A managed database service:

| Setting | Value | Rationale |
| -------------------- | ------------------------------------ | ---------------------------------------- |
| Engine | PostgreSQL | JSONB support, robust indexing |
| Instance type | Memory-optimized | Reporting queries benefit from memory |
| Multi-AZ | Enabled | Automatic failover |
| Read replicas | 1 (for reporting) | Isolates analytical from transactional |
| Backup retention | 30 days | Compliance requirement |
| Deletion protection | Enabled | Prevents accidental deletion |

**Multi-AZ failover:** In case of primary failure, the database automatically fails over to a standby. The DNS endpoint does not change, so the application reconnects without configuration changes.

**Read replicas for reporting:** Complex aggregation queries (P&L reports, forecasting) are routed to a read replica, preventing them from competing with customer-facing API requests for resources.

### Cache Layer

A managed cache service handles caching, rate limiting, and job queuing:

| Use Case | Strategy | Rationale |
| --------------------- | --------------------------------- | ------------------------------------------- |
| API response cache | Cache-aside with TTL | Reduces database load |
| Rate limiting | Sliding window | Sub-second precision without DB writes |
| Job queue | Queue operations | Simple, reliable |
| Session cache | Key-value with TTL | JWT is client-side, so not primary session store |

**Rate limiting:** Each user or API key gets a sorted set in the cache. Timestamps of recent requests are added. Entries outside the window are removed, and the remaining count determines if the request is allowed. This gives precise sliding window rate limiting.

### File Storage

File storage holds generated reports, exported files, and uploaded assets:

| Bucket | Purpose | Access Control |
| ------------------- | ------------------------------------ | --------------------------------------- |
| reports-storage | Generated reports | Presigned URLs with time-limited access |
| exports-storage | Downloaded files | Direct access via CDN |
| logs-storage | Access logs, monitoring exports | Internal only |

**Presigned URLs:** Generated reports are stored with server-side encryption. When a user requests a download, the backend generates a time-limited URL that provides temporary access without making the storage public.

---

## Scaling Strategy

| Resource | Scaling Strategy | Trigger |
| -------------------- | --------------------------------------------------------------------------------- | ------------------------------------------ |
| Frontend delivery | CDN edge caching. Scales automatically | Traffic increases (no action needed) |
| Backend API | Auto-scaling when CPU exceeds threshold for sustained period | CPU utilization, request latency |
| Database | Vertical scaling (larger instance). Read replicas for reporting | Connection count, query queue depth |
| Cache | Vertical scaling (larger node). Cluster mode if needed | Memory usage threshold |
| Background workers | Queue depth triggers additional workers | Queue size threshold |

### Auto-Scaling Configuration

The auto-scaling group uses target tracking:
- Scale-out when CPU exceeds 70% for sustained period
- Scale-in after cooldown period
- Cooldown prevents rapid scaling fluctuations

### Database Scaling

The database starts at a baseline instance and can scale vertically. Read replicas handle reporting queries — the application routes read-only queries to replicas and write queries to the primary. This prevents heavy aggregation queries from affecting transactional performance.

---

## Security Controls

### Network Security

```
Internet
    |
CDN (WAF enabled)
    |
Load Balancer (SSL termination)
    |
Security Group: API (inbound from load balancer only)
    |
Database Security Group (inbound from API SG only)
    |
Cache Security Group (inbound from API SG only)
```

- **Defense in depth**: CDN provides edge-level protection (WAF). Load balancer terminates SSL. Security groups restrict access between layers. Database and cache are in private subnets with no direct internet access
- **CORS**: Restricted to the application's domain
- **All traffic is HTTPS**: No plain HTTP traffic reaches the application

### Data Protection

| Data | Protection Mechanism |
| ---------------- | ------------------------------------------------------------------------- |
| In transit | TLS 1.2+ for all external and internal traffic |
| At rest (Database) | Encryption at rest |
| At rest (Storage) | Server-side encryption |
| Secrets | Managed secrets service for API keys, credentials |
| Client tokens | Client-side encryption for sensitive stored data |

### Secrets Management

Sensitive configuration values are stored in a managed secrets service:
- Database credentials (rotated periodically)
- External API client IDs and secrets
- Encryption keys
- Service account keys

The application retrieves secrets at startup and caches them in memory.

---

## Monitoring and Observability

| Tool | Purpose | Key Metrics |
| ----------------- | ----------------------------------------- | -------------------------------------------- |
| Infrastructure monitoring | Resource utilization | CPU, memory, disk, network, error rates |
| Database performance insights | Query performance | Slow queries, wait events, connection count |
| Access logs | Request-level logging | Request path, status, latency, user agent |
| Application logs | Structured logging | Error rates, response times, endpoint usage |

### Alert Thresholds

| Condition | Alert Level |
| --------------------------------------- | ----------- |
| API latency exceeds threshold | Warning |
| Error rate exceeds threshold | Critical |
| Connection count near maximum | Warning |
| Certificate expires within warning period | Info |

---

## Cost Optimization

| Area | Strategy | Estimated Savings |
| ----------------- | ---------------------------------------------------------- | ----------------- |
| Compute | Burstable instances. Auto-scaling stops unused instances | 30-40% vs always-on |
| Storage | General purpose volumes (baseline IOPS included) | 20% vs provisioned |
| Data transfer | CDN reduces origin load. Compression | 50-60% vs direct |
| Database | Read replicas for reporting | 50% vs larger primary |
| Reserved instances | Reserved capacity for baseline | 40% vs on-demand |

---

## Interview Talking Points

**On the static-first approach:** "Serving the React app as static files from a CDN means zero server load for page delivery. Users anywhere in the world get files from the nearest edge location. The backend only handles API requests, not HTML rendering. This also means the frontend can be deployed independently."

**On the caching strategy:** "Static assets are named with content hashes. When we deploy a new version, the CDN sees new file names and serves updated assets immediately. Old cached files are never served with stale content. This eliminates cache invalidation, which is one of the hardest problems in CDN-based deployments."

**On the database read replica for reporting:** "Complex aggregation queries like P&L reports can scan millions of rows. Running these on the primary database would impact customer-facing API responses. We route all reporting queries to a read replica, isolating transactional from analytical workloads."

**On the rate limiting implementation:** "We use the cache's sorted set operations for rate limiting. Each API request adds its timestamp to a sorted set and atomically removes entries outside the window. This gives sub-second precision for rate limit enforcement without adding load to the database."

---

## Related Documents

- [Deployment Pipeline](../deployment/deployment-pipeline.md) - CI/CD process and automation
- [Database Design](../database/schema-architecture.md) - Data storage architecture
- [Security Architecture](../security/security-overview.md) - Authentication and encryption
- [Backend Service Architecture](../backend/service-architecture.md) - Application layer design
