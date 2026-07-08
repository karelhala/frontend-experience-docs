# Environments, CI/CD & Repository Map

## Environment Matrix

```
┌───────────────┬─────────────────────────────┬─────────────────────┬──────────┐
│ Environment   │ Hostname                    │ SSO                 │ Purpose  │
├───────────────┼─────────────────────────────┼─────────────────────┼──────────┤
│ Production    │ console.redhat.com          │ sso.redhat.com      │ Users    │
│ Stage         │ stage.console.redhat.com    │ sso.stage.redhat.com│ QA       │
│ QA            │ qa.console.redhat.com       │ sso.qa.redhat.com   │ QA       │
│ CI            │ ci.console.redhat.com       │ sso.qa.redhat.com   │ CI/CD    │
│ FRH Stage     │ stage.openshiftusgov.com    │ sso.stage.openshift │ GovCloud │
│               │                             │ usgov.com           │          │
│ FRH Prod      │ console.openshiftusgov.com  │ sso.openshiftusgov  │ GovCloud │
│               │                             │ .com                │          │
│ Ephemeral     │ *.ephemeral.cluster         │ sso.stage.redhat.com│ PR tests │
│ Dev           │ localhost:1337               │ sso.stage.redhat.com│ Local    │
├───────────────┼─────────────────────────────┼─────────────────────┼──────────┤
│ Clusters      │ CRCD01UE1, CRCS02UE1,       │                     │ ROSA     │
│ (multi)       │ HCCP01UE1, CRCP01UE1        │                     │ clusters │
└───────────────┴─────────────────────────────┴─────────────────────┴──────────┘

ITLess environments: FRH Stage, FRH Prod, Ephemeral, Internal, Secure Cloud
(No IT-managed user accounts — different auth flow)
```

---

## CI/CD Pipeline

```
┌─── Developer Workflow ────────────────────────────────────────────┐
│                                                                    │
│  1. Push to feature branch                                         │
│       │                                                            │
│       ▼                                                            │
│  2. GitHub Actions CI (shared-workflows)                           │
│     ├── Lint (ESLint, shared config)                               │
│     ├── Type check (TypeScript)                                    │
│     ├── Unit tests (Jest)                                          │
│     ├── Build (Webpack)                                            │
│     └── SC Environment Impact Check                                │
│         (detects DB migrations, ClowdApp changes, Kafka topics)    │
│       │                                                            │
│       ▼                                                            │
│  3. PR Review + Merge                                              │
│       │                                                            │
│       ▼                                                            │
│  4. Konflux Build Pipeline                                         │
│     ├── Build container image                                      │
│     ├── Push to Quay registry (with digest)                        │
│     ├── Security scan (Clair, ACS)                                 │
│     └── Sign image (cosign)                                        │
│       │                                                            │
│       ▼                                                            │
│  5. App-Interface (GitOps)                                         │
│     ├── Update image digest in saas-deploy config                  │
│     ├── Deploy to stage cluster                                    │
│     └── Promote to production (manual approval)                    │
│       │                                                            │
│       ▼                                                            │
│  6. Frontend-Operator Reconciliation                               │
│     ├── Detect image change in Frontend CRD                        │
│     ├── Rolling update Deployment                                  │
│     ├── Run valpop push-cache job                                  │
│     ├── Run Akamai cache-bust job                                  │
│     └── Update aggregated ConfigMaps                               │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### Shared Workflows (shared-workflows repo)

| Workflow | Purpose |
|----------|---------|
| `stale.yml` | Mark and close stale issues/PRs after configurable inactivity |
| `sc-environment-impact-check.yml` | Analyze PR impact on SC Environment deployments |
| Renovate presets (frontend) | npm dependency management with grouped updates |
| Renovate presets (backend) | Go module + UBI9 base image pinning |
| PR templates | Accessibility, screenshot, OUIA testing checklists |

---

## Repository Map

```
Console Framework Team — Repository Ownership

Shell & Framework
├── insights-chrome              TypeScript/React  Application shell
├── chrome-service-backend       Go                Shell backend
├── frontend-operator            Go                K8s operator
├── frontend-components          TypeScript        Shared component monorepo (15 pkgs)
├── frontend-assets              HTML/CSS          Shared static assets
└── widget-layout                TypeScript/React  Dashboard widget system

Applications
├── learning-resources           TypeScript/React  Learning content
├── service-accounts             TypeScript/React  Service account management
├── astro-virtual-assistant-frontend  TypeScript   AI chat widget
├── scheduler-ui                 TypeScript/React  Report scheduling
├── frontend-starter-app         TypeScript/React  App template
├── landing-page-frontend        TypeScript/React  Console home page
├── insights-rbac-ui             TypeScript/React  RBAC management
├── notifications-frontend       TypeScript/React  Notification config
├── access-requests-frontend     TypeScript/React  TAM access tool
├── user-preferences-frontend    JavaScript/React  User settings
├── api-frontend                 TypeScript/React  API docs (legacy)
├── api-documentation-frontend   TypeScript/React  API docs (current)
├── sources-ui                   JavaScript/React  Integrations
├── subscriptions-dashboard-ui   TypeScript/React  Subscriptions
└── payload-tracker-frontend     JavaScript/React  Payload tracking

Backend Services
├── quickstarts                  Go                Quickstart content
├── pdf-generator                TypeScript/Node   PDF report generation
├── widget-layout-backend        Go                Widget persistence
├── dynamic-browser-events       Go                WebSocket events
└── astro-virtual-assistant-v2   Python            VA backend

Infrastructure & Tooling
├── frontend-asset-proxy         Go/Caddy          Asset reverse proxy
├── valpop                       Go                Asset cache (ValKey)
├── frontend-development-proxy   Go/Caddy          Local dev proxy
├── insights-frontend-builder-common  Python/Shell Build pipeline
├── shared-workflows             GitHub Actions    Shared CI workflows
├── javascript-clients           TypeScript        Generated API clients
├── javascript-clients-tests     TypeScript        API client tests
├── frontend-test-utils          TypeScript        Playwright helpers
├── cypress-e2e-image            Dockerfile        Cypress test image
├── hcc-storybook-hub            TypeScript        Storybook aggregator
├── frontend-experience-docs     Markdown          Team documentation
└── consoles-enhancements        GitHub Issues     Enhancement tracking

AI & Agentic Systems
├── ai-web-clients               TypeScript/NX     AI client libraries (5 pkgs)
├── platform-frontend-ai-toolkit TypeScript        AI dev tools & skills
├── platform-frontend-ai-dev     Python            Dev Bot (Rehor)
├── hcc-framework-agent-dev      Shell             Framework team bot runner
├── hcc-ui-agent-dev             Shell             UI team bot runner
└── hcc-ai-assistant             Python            Production AI service
```

---

## Platform Scale (July 2026)

```
┌────────────────────────────────────────────────┐
│  Repositories maintained          40+           │
│  Federated micro-frontends        30+           │
│  npm packages published           20+           │
│  Navigation bundles               12            │
│  CRD types                        3             │
│  K8s clusters (multicluster)      4+            │
│  Deployment environments          10+           │
│  Chrome API methods               ~60           │
│  Dashboard widget types           13+           │
│  AI client libraries              5             │
│  PatternFly version               6.x           │
│  React version                    18.3.x        │
│  Webpack Module Federation        5.x           │
│  Scalprum (MFE orchestrator)      0.11.x        │
│  Feature flag system              Unleash        │
│  Analytics                        Amplitude +    │
│                                   Segment +      │
│                                   Pendo          │
│  Team size (Framework)            ~10            │
│  Bot personas                     8              │
└────────────────────────────────────────────────┘
```
