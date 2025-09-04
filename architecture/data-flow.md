# Data Flow

This document traces how data moves through the system for several key workflows. Understanding these flows helps you see how the frontend, backend, Amazon APIs, and third-party services interact.

---

## What You Will Learn

- How data flows during Amazon account authorization
- How the PPC strategy engine evaluates and adjusts campaigns
- How the async report generation pipeline works
- How real-time notifications reach the user
- How data synchronization keeps things in sync with Amazon

---

## Amazon Account Authorization Flow

This flow happens when a user authorizes their Amazon account for the first time.

```mermaid
sequenceDiagram
    actor User
    participant Frontend
    participant Backend
    participant AmazonLWA
    participant AmazonAPI

    User->>Frontend: Click "Authorize Account"
    Frontend->>Backend: Request authorization URL
    Backend-->>Frontend: Redirect URL for Amazon LWA
    Frontend->>AmazonLWA: Redirect user to Amazon login
    User->>AmazonLWA: Enter Amazon credentials
    AmazonLWA-->>User: Grant permissions page
    User->>AmazonLWA: Approve SP-API and Ads permissions
    AmazonLWA-->>Frontend: Redirect with auth code
    Frontend->>Backend: Send auth code
    Backend->>AmazonLWA: Exchange code for refresh token
    AmazonLWA-->>Backend: Return refresh token
    Backend->>Backend: Store refresh token securely
    Backend-->>Frontend: Authorization success
    Frontend-->>User: Account authorized, starting sync
    Backend->>AmazonAPI: Start data sync with access tokens
    AmazonAPI-->>Backend: Return inventory, sales, ad data
    Backend->>Backend: Process and store data
    Backend-->>Frontend: Sync progress updates
    Frontend-->>User: Dashboard shows synced data
```

Key points about this flow:

- The frontend never sees the refresh token. It is exchanged and stored server-side.
- Amazon's LWA handles all credential collection. The platform never touches the user's Amazon password.
- The sync starts automatically after authorization. The user does not need to trigger it manually.

---

## PPC Strategy Evaluation Flow

This flow runs periodically to evaluate and adjust PPC campaigns based on strategy rules.

```mermaid
sequenceDiagram
    participant StrategyEngine
    participant AmazonAPI
    participant Database
    participant Frontend

    StrategyEngine->>Database: Load active strategies
    Database-->>StrategyEngine: List of strategies with rules

    loop Each Strategy
        StrategyEngine->>Database: Get campaigns linked to strategy
        Database-->>StrategyEngine: Campaign list

        loop Each Campaign
            StrategyEngine->>AmazonAPI: Pull latest performance data
            AmazonAPI-->>StrategyEngine: Impressions, clicks, spend, sales, ACOS

            StrategyEngine->>StrategyEngine: Compare performance against targets

            alt ACOS exceeds target
                StrategyEngine->>StrategyEngine: Calculate bid reduction
                StrategyEngine->>AmazonAPI: Update campaign bid
                AmazonAPI-->>StrategyEngine: Bid updated
                StrategyEngine->>Database: Log the change
            end

            alt ACOS below target (opportunity)
                StrategyEngine->>StrategyEngine: Calculate bid increase
                StrategyEngine->>AmazonAPI: Update campaign bid
                AmazonAPI-->>StrategyEngine: Bid updated
                StrategyEngine->>Database: Log the change
            end

            alt Campaign underperforming
                StrategyEngine->>StrategyEngine: Mark for pause
                StrategyEngine->>AmazonAPI: Pause campaign
                AmazonAPI-->>StrategyEngine: Campaign paused
                StrategyEngine->>Database: Log the pause action
            end

            alt High-performing search term found
                StrategyEngine->>StrategyEngine: Prepare keyword add
                StrategyEngine->>AmazonAPI: Add as targeting keyword
                AmazonAPI-->>StrategyEngine: Keyword added
                StrategyEngine->>Database: Log the keyword addition
            end
        end
    end

    StrategyEngine-->>Frontend: Update available via polling
    Frontend->>Database: Fetch updated history
    Database-->>Frontend: History with all changes
    Frontend-->>User: Show updated strategy status
```

The strategy engine runs on a schedule in the background. It does not wait for user interaction. Every change it makes is logged with the before and after state, so users can audit what happened.

---

## Async Report Generation Flow

Some reports take too long to generate within a single HTTP request. This flow handles that case.

```mermaid
sequenceDiagram
    actor User
    participant Frontend
    participant API
    participant JobQueue
    participant ReportEngine
    participant Database

    User->>Frontend: Click "Generate All Products Events Report"
    Frontend->>API: POST /api/v1/futuristics/reports
    API->>JobQueue: Enqueue report generation job
    API-->>Frontend: Return job ID, status: pending
    Frontend->>Frontend: Start polling service
    Frontend->>Frontend: Show "Generating" toast notification

    JobQueue->>ReportEngine: Process report job
    ReportEngine->>Database: Query futuristic data for all products
    Database-->>ReportEngine: Raw data
    ReportEngine->>ReportEngine: Aggregate events by date and type
    ReportEngine->>Database: Store completed report

    loop Every 30 seconds, up to 5 times
        Frontend->>API: GET /api/v1/futuristics/reports/latest
        API->>Database: Check report status
        Database-->>API: Status: completed
        API-->>Frontend: Report is ready
    end

    Frontend->>Frontend: Stop polling
    Frontend->>Frontend: Update toast to "Report Ready" with "View" button
    Frontend-->>User: Toast notification appears

    User->>Frontend: Click "View Report"
    Frontend->>Frontend: Navigate to report page with report data
    Frontend-->>User: Shows all products events calendar
```

If the user navigates away from the page while a report is generating, the polling continues. When the report is ready, a toast notification appears on whatever page they are on. Clicking the toast navigates to the report view.

---

## Real-Time Notification Flow

Push notifications reach the user through two channels: browser push notifications (even when the app is backgrounded) and in-app notifications in the notification panel.

```mermaid
sequenceDiagram
    participant Backend
    participant Firebase
    participant ServiceWorker
    participant Frontend
    participant Firestore
    actor User

    Note over Frontend: App initialization
    Frontend->>Firebase: Request notification permission
    User->>Frontend: Grant permission
    Frontend->>Firebase: Get FCM device token
    Firebase-->>Frontend: Return token
    Frontend->>Backend: POST /api/v1/auth/save-fcm-token
    Note over Frontend,Backend: Body: { token: "...", platform: "web" }
    Backend->>Backend: Store token for this user

    Note over Backend,Firebase: When an event occurs
    Backend->>Firebase: Send push notification via FCM
    Firebase->>ServiceWorker: Deliver push payload

    alt App is in background
        ServiceWorker->>ServiceWorker: Show browser notification
        ServiceWorker-->>User: Native browser notification
        User->>Frontend: Click notification
        Frontend->>Frontend: Navigate to relevant page
    else App is in foreground
        ServiceWorker->>Frontend: onMessage handler fires
        Frontend->>Frontend: Show in-app notification or browser notification
        Frontend-->>User: Notification appears in-app
    end

    Note over Backend,Firestore: Also write to Firestore for panel
    Backend->>Firestore: Update notification summary document
    Firestore-->>Frontend: Real-time update via onSnapshot
    Frontend->>Frontend: Update badge count
    Frontend-->>User: Badge count updates immediately
```

Two channels exist because browser push notifications work even when the tab is in the background, while Firestore provides instant in-app updates when the user is actively using the platform.

---

## Data Synchronization Flow

When Amazon data changes, the platform detects and syncs the changes.

```mermaid
sequenceDiagram
    participant SyncScheduler
    participant Backend
    participant AmazonAPI
    participant Database
    participant Frontend

    SyncScheduler->>Backend: Trigger sync cycle

    Backend->>Backend: Determine which steps need syncing
    Backend->>Database: Update sync log status to "running"

    par Sync Step: Inventory
        Backend->>AmazonAPI: GET /fba/inventory/summaries
        AmazonAPI-->>Backend: Inventory data
        Backend->>Database: Update inventory records
        Backend->>Database: Mark inventory_list sync complete
    and Sync Step: Products
        Backend->>AmazonAPI: GET /catalog/items
        AmazonAPI-->>Backend: Product details
        Backend->>Database: Update product records
        Backend->>Database: Mark product_list sync complete
    and Sync Step: Sales
        Backend->>AmazonAPI: GET /sales/metrics
        AmazonAPI-->>Backend: Sales data
        Backend->>Database: Update sales records
        Backend->>Database: Mark sales sync complete
    end

    Backend->>Database: Update sync log status to "completed"

    Frontend->>Backend: GET /api/v1/auth/sync-status
    Backend-->>Frontend: Sync status with step details
    Frontend->>Frontend: Update sidebar sync icons
    Frontend-->>User: Sync icons disappear as steps complete
```

The sync runs automatically on a schedule and can also be triggered manually from the settings page. If a sync step fails (for example, Amazon API returns an error), that step is retried on the next cycle. Completed steps are not reprocessed.

---

## Related Documents

- [Architecture Overview](overview.md) - System-level architecture
- [Multi-Tenant Design](multi-tenant-design.md) - Data isolation across marketplaces
- [Authentication Flow](../authentication/authentication-flow.md) - User authentication
- [Report Polling](../notifications/report-polling.md) - Async report generation in detail
