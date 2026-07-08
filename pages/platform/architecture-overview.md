# Architecture Overview

The Red Hat Hybrid Cloud Console (`console.redhat.com`) is a unified web platform serving Red Hat's cloud product portfolio — RHEL Insights, OpenShift, Ansible Automation Platform, Application Services, and more. It is built as a **micro-frontend architecture** where 30+ independent applications are loaded into a shared shell at runtime.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        console.redhat.com                           │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                    insights-chrome (Shell)                     │  │
│  │  ┌─────────┐ ┌──────────┐ ┌────────┐ ┌───────┐ ┌─────────┐  │  │
│  │  │Masthead │ │Navigation│ │  Auth  │ │Search │ │Analytics│  │  │
│  │  └─────────┘ └──────────┘ └────────┘ └───────┘ └─────────┘  │  │
│  │  ┌───────────────────────────────────────────────────────┐    │  │
│  │  │              Content Area (Federated Apps)             │    │  │
│  │  │                                                       │    │  │
│  │  │  ┌─────────┐ ┌──────────┐ ┌───────┐ ┌────────────┐   │    │  │
│  │  │  │Advisor  │ │Compliance│ │ RBAC  │ │Vuln Mgmt   │   │    │  │
│  │  │  └─────────┘ └──────────┘ └───────┘ └────────────┘   │    │  │
│  │  │  ┌─────────┐ ┌──────────┐ ┌───────┐ ┌────────────┐   │    │  │
│  │  │  │Sources  │ │Notif.    │ │Landing│ │Subscriptions│  │    │  │
│  │  │  └─────────┘ └──────────┘ └───────┘ └────────────┘   │    │  │
│  │  │               ... 30+ applications ...                │    │  │
│  │  └───────────────────────────────────────────────────────┘    │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

## Key Numbers

| Metric | Value |
|--------|-------|
| Federated applications | 30+ |
| Shared component packages | 15 |
| Navigation bundles | 12 (RHEL, OpenShift, Ansible, IAM, Settings, ...) |
| Deployment environments | 10+ (prod, stage, QA, CI, FRH, ephemeral, ...) |
| Chrome API methods | ~60 |
| Widget types | 13+ |
| AI client libraries | 5 |

---

## High-Level Architecture

```
                                    ┌──────────────┐
                                    │   Browser    │
                                    └──────┬───────┘
                                           │
                                    ┌──────▼───────┐
                                    │  Akamai CDN  │
                                    │  (edge cache)│
                                    └──────┬───────┘
                                           │
                          ┌────────────────┼────────────────┐
                          │                │                │
                   ┌──────▼──────┐  ┌──────▼──────┐  ┌─────▼──────┐
                   │  frontend-  │  │   chrome-   │  │  3scale /  │
                   │ asset-proxy │  │  service-   │  │  Gateway   │
                   │  (Caddy)    │  │  backend    │  │  (API)     │
                   └──────┬──────┘  └──────┬──────┘  └─────┬──────┘
                          │                │               │
                   ┌──────▼──────┐  ┌──────▼──────┐       │
                   │  S3/MinIO   │  │ PostgreSQL  │       │
                   │  (assets)   │  │ (user data) │       │
                   └─────────────┘  └─────────────┘       │
                                                          │
                   ┌──────────────────────────────────────┘
                   │
            ┌──────▼──────────────────────────────────────────────┐
            │              Kubernetes Cluster (ROSA)               │
            │                                                      │
            │  ┌──────────────────────────────────────────────┐    │
            │  │          frontend-operator (Go)               │    │
            │  │  Watches: Frontend, FrontendEnvironment,      │    │
            │  │           Bundle CRDs                         │    │
            │  │  Creates: Deployments, Services, Ingresses,   │    │
            │  │           ConfigMaps, ServiceMonitors         │    │
            │  └──────────────────────────────────────────────┘    │
            │                                                      │
            │  ┌──────────┐ ┌──────────┐ ┌──────────┐            │
            │  │chrome    │ │advisor   │ │compliance│  ...30+     │
            │  │frontend  │ │frontend  │ │frontend  │  pods       │
            │  └──────────┘ └──────────┘ └──────────┘            │
            └──────────────────────────────────────────────────────┘
```

### Infrastructure Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| CDN | Akamai | Edge caching, DDoS protection, cache busting |
| Reverse Proxy | Caddy (frontend-asset-proxy) | Asset serving from S3/MinIO, SPA routing |
| Shell Backend | Go (chrome-service-backend) | User prefs, nav config, search index, WebSocket |
| App Shell | TypeScript/React (insights-chrome) | Auth, navigation, theming, module loading |
| Micro-frontends | TypeScript/React (per-app) | Individual product UIs |
| Orchestration | Go (frontend-operator) | K8s operator managing Frontend CRDs |
| Asset Cache | Go (valpop) + ValKey/Redis | Asset versioning, rollback |
| Feature Flags | Unleash | Per-user/org feature toggles |
| Analytics | Amplitude, Segment, Pendo | Usage tracking, feedback |
| Monitoring | Prometheus + ServiceMonitor | Metrics, alerting |

---

## Request Flow — From Browser to Application

```
User navigates to console.redhat.com/insights/advisor
                │
                ▼
┌─── 1. DNS + Akamai ────────────────────────────────────────────────┐
│  Akamai resolves, checks edge cache.                               │
│  Cache HIT  → serve cached assets directly                        │
│  Cache MISS → forward to origin                                    │
└────────────────────────────┬───────────────────────────────────────┘
                             ▼
┌─── 2. Origin Routing ─────────────────────────────────────────────┐
│  /apps/chrome/*     → chrome frontend pod (insights-chrome)        │
│  /apps/advisor/*    → advisor frontend pod                         │
│  /api/chrome-service/* → chrome-service-backend                    │
│  /api/*             → 3scale gateway → backend services            │
└────────────────────────────┬───────────────────────────────────────┘
                             ▼
┌─── 3. Shell Bootstrap (insights-chrome) ──────────────────────────┐
│                                                                    │
│  a. Load chrome entry point (chrome-root.[hash].js)                │
│  b. Initialize Keycloak SSO → obtain JWT token                     │
│  c. Fetch user config from /api/chrome-service/v1/user             │
│  d. Fetch feature flags from /api/featureflags/v0                  │
│  e. Fetch fed-modules manifest (module registry)                   │
│  f. Fetch navigation bundles (bundles-generated.json)              │
│  g. Render shell: masthead, sidebar, content area                  │
│  h. Initialize analytics (Amplitude, Segment, Pendo)               │
│  i. Connect WebSocket for live notifications                       │
│                                                                    │
└────────────────────────────┬───────────────────────────────────────┘
                             ▼
┌─── 4. Application Loading (Scalprum + Module Federation) ─────────┐
│                                                                    │
│  a. Match URL pathname to module route in fed-modules manifest     │
│  b. Resolve federated module scope (e.g., "advisor")               │
│  c. Load remote entry from /apps/advisor/fed-mods.json             │
│  d. Webpack Module Federation resolves shared deps (React, PF)     │
│  e. Scalprum renders <ScalprumComponent> in content area           │
│  f. App receives useChrome() context (auth, nav, RBAC, etc.)       │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### Timing Breakdown (typical)

```
0ms    ─── Akamai edge cache hit (static assets)
50ms   ─── Chrome JS bundle loaded
150ms  ─── Keycloak SSO token obtained
200ms  ─── User config + feature flags fetched (parallel)
250ms  ─── Navigation + module manifest fetched (parallel)
300ms  ─── Shell rendered (masthead, sidebar visible)
400ms  ─── App remote entry loaded
500ms  ─── App component rendered in content area
         ─── WebSocket connected (async, non-blocking)
```
