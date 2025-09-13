# Deployment Pipeline

The application uses a CI/CD pipeline for continuous integration and deployment. This document covers the pipeline architecture, build optimization strategies, deployment approaches, rollback strategy, environment management, and operational runbooks.

---

## What You Will Learn

- How the CI/CD pipeline is structured from commit to production
- How builds are optimized for speed and reliability
- How deployment strategies provide zero-downtime considerations
- How rollbacks are triggered and executed
- How environment variables and secrets are managed across environments
- How health checks validate deployment success
- How monitoring and alerting integrate with the pipeline

---

## Pipeline Architecture

```
Git Push (Branch: staging)
    |
    v
Build Stage (Build & Test)
    |
    |-- Install dependencies (cached)
    |-- Run unit and integration tests
    |-- Run production build
    |-- Upload artifact to storage
    v
Deploy Stage (Deploy to server)
    |
    |-- Download artifact from storage
    |-- Run pre-deployment hooks
    |-- Copy files to target directory
    |-- Run post-deployment hooks
    |-- Start server
    v
Health Check Validation
    |
    |-- Check health endpoint (expect 200)
    |-- Validate response structure
    v
Deployment Complete
    |
    |-- Success: Traffic routes to new instance
    |-- Failure: Automatic rollback to previous version
```

---

## Stage 1: Build Stage

### Build Configuration

The build configuration defines the environment, commands, and artifacts:

The build phase runs sequentially: install dependencies (using lockfile for deterministic installs), run unit and integration tests, run the production build, and create a compressed deployment artifact.

### Build Optimization

| Optimization | Technique | Impact |
| ----------------------- | --------------------------------------------------------- | ------------------------- |
| Dependency caching | Use lockfile with prefer-offline mode | Reduces install time significantly |
| Artifact compression | Compress build output before upload | Reduces upload time |
| Parallel test execution | Run test files in parallel | Reduces test time |
| Build output analysis | Log bundle size for monitoring | Prevents bundle bloat |
| Conditional builds | Skip build on docs-only changes | Saves compute time |

### Build Failure Handling

If the build fails at any phase:
1. **Install failure**: Check registry availability, retry with clean cache
2. **Test failure**: Full test output logged, build stops and notifies the team
3. **Build failure**: Usually syntax error or missing dependency — log shows exact error
4. **Artifact failure**: Rare, usually disk space or permissions issue

---

## Stage 2: Deploy Stage

### Deployment Configuration

The deployment configuration defines how the build artifact is deployed to the server:

The deploy phase hooks handle: gracefully stopping the current server before install, extracting the artifact and setting permissions after install, starting the server in production mode, and validating the deployment with health check retries.

### Deployment Script Responsibilities

**Before Install:**
- Stop the current server gracefully (handle case where no server is running)
- Wait for process to release the port
- Clear the deployment directory

**After Install:**
- Extract the deployment artifact
- Set appropriate file permissions
- Make scripts executable

**Application Start:**
- Start the server in production mode on the configured port
- Use process management to ensure the process survives the script
- Store the PID for monitoring

**Service Validation:**
- Perform health check by hitting the local server
- Retry with configurable attempts and interval
- Exit with error if health check fails all attempts

---

## Stage 3: Health Check and Validation

### What the Health Check Validates

The health check verifies:
1. **HTTP 200 response** — Server is running and serving requests
2. **Response time within threshold** — App is not overloaded
3. **Correct content type** — Response includes expected headers
4. **Non-empty body** — Response has content (not a blank page)

### Post-Deployment Smoke Tests

After the health check passes, an automated smoke test suite runs:
- Verify static assets are served correctly
- Verify SPA routing works (deep links return index.html)
- Verify API connectivity
- Verify critical endpoints return expected responses

If any smoke test fails, the deployment is flagged for investigation.

---

## Rollback Strategy

### Automatic Rollback Triggers

Rollbacks are automatically triggered when:

| Condition | Detection Method | Response Time |
| ---------------------------- | --------------------------------------- | --------------- |
| Health check failure | Validation script returns non-zero | ~30 seconds |
| Deployment hook timeout | Script exceeds timeout limit | ~5 minutes |
| Application crash loop | Server process dies repeatedly | ~2 minutes |
| Error rate spike | Monitor detects elevated error rate | ~5 minutes |

### Rollback Execution

When a rollback is triggered:
1. Current deployment is marked as "failed"
2. Previous deployment artifact is retrieved from storage
3. Same deployment lifecycle runs with the previous artifact
4. If rollback succeeds, deployment is marked as "rolled back"
5. If rollback fails, the incident escalates to manual intervention

### Manual Rollback

A manual rollback can be triggered through the deployment console or CLI:
- Stop the current deployment
- Redeploy the last successful version
- Verify health checks pass

### Post-Rollback Actions

After a rollback completes:
1. Notification sent to the engineering team
2. Rollback logged with reason and timing
3. Failed artifact preserved for debugging
4. Root cause investigation initiated

---

## Environment Management

### Environment Hierarchy

```
local --> development --> staging --> production
```

Each environment has its own:
- Build configuration with environment-specific variables
- Deployment application with environment-specific settings
- Storage for deployment artifacts
- Monitoring dashboard with environment-specific alarms

### Environment Configuration

| Variable | Development | Staging | Production |
| ------------------------------- | ---------------------- | ---------------------- | ---------------------- |
| API base URL | localhost | staging-api | production-api |
| API keys | Test credentials | Test credentials | Live credentials |
| Session timeout | Longer (debugging) | Standard | Standard |

### Promotion Strategy

Code promotions follow a strict path:
```
Feature Branch --> Staging (auto-deploy) --> Production (manual approve)
```

1. **Feature branches**: Auto-deployed to preview or tested locally
2. **Staging branch**: Auto-deployed on push. Used for QA and integration testing
3. **Production**: Manual approval required. Only staging-tested code is promoted

---

## Build Artifact Storage

### Artifact Lifecycle

| Stage | Storage Location | Retention | Purpose |
| ---------------------- | ---------------------- | ------------ | ------------------------------ |
| Build artifact | Build storage | 30 days | Deployment and rollback |
| Failed build artifact | Build storage | 90 days | Debugging failed deployments |
| Previous deployment | Build storage | Until replaced | Rollback target |

### Artifact Naming Convention

Artifacts include the environment, date, time, and git commit hash in the filename. This makes it easy to identify which version is deployed and trace it back to the source code.

---

## Monitoring and Observability

### Deployment Dashboard

A dashboard shows:
- **Deployment status** — Success, failed, or in-progress for recent deployments
- **Build duration** — Time per build phase
- **Deployment duration** — Time per deployment hook
- **Rollback frequency** — How often and why
- **Environment versions** — Currently deployed version per environment

### Alarms

| Alarm | Condition | Action |
| ------------------------ | ------------------------------------------- | --------------------------------- |
| Deployment failed | Deployment status = failed | Notification to team |
| Build failed | Build status = failed | Notification to committer |
| Rollback triggered | Rollback event | Alert (production only) |
| Health check flapping | Alternating success/failure | Alert |

---

## Interview Talking Points

**On the health check design:** "The health check validates that the server is up and responding correctly. It retries with configurable intervals to handle transient startup delays. If all retries fail, the deployment system triggers an automatic rollback. This catches failed deployments quickly and prevents users from seeing errors."

**On the rollback strategy:** "Rollbacks are handled automatically. When a deployment fails health checks, the previous artifact is redeployed through the same lifecycle hooks. The key insight is that deploy scripts are designed to be idempotent — running pre-deployment on the rollback artifact is the same as running it on the new artifact."

**On the artifact naming convention:** "Build artifacts include the date, time, and git commit hash. When a deployment is rolled back, we can tell exactly which version was deployed and trace it back to the source code."

**On the environment promotion strategy:** "Code flows from feature branches to staging to production. Staging auto-deploys on merge. Production requires manual approval. This means staging catches integration issues before they reach production, while the manual gate prevents accidental production deployments."

---

## Related Documents

- [Cloud Infrastructure](../infrastructure/cloud-architecture.md) - Cloud architecture and networking
- [Database Design](../database/schema-architecture.md) - Data storage architecture
- [Testing Strategy](../testing/testing-strategy.md) - Testing approach
