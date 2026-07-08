# Frontend Operator — Kubernetes-Native Deployment

**Repository:** [RedHatInsights/frontend-operator](https://github.com/RedHatInsights/frontend-operator)  
**Technology:** Go, Kubebuilder, Kubernetes Operator SDK  
**Role:** Watches Custom Resource Definitions (CRDs) and reconciles the desired state of all frontend deployments.

## Custom Resource Definitions

```
┌─────────────────────────────────────────────────────────────┐
│                    CRD Hierarchy                             │
│                                                              │
│  FrontendEnvironment (1 per environment)                     │
│  ├── hostname: stage.console.redhat.com                      │
│  ├── sso: https://sso.stage.redhat.com                       │
│  ├── bundles: [insights, openshift, ansible, ...]            │
│  ├── serviceCategories: [{id, title, groups}]                │
│  ├── enableAkamaiCacheBust: true                             │
│  ├── enablePushCache: true                                   │
│  ├── targetNamespaces: [ns1, ns2, ...]                       │
│  └── defaultReplicas: 1                                      │
│       │                                                      │
│       ├── Frontend (1 per application)                       │
│       │   ├── envName: "stage"                               │
│       │   ├── title: "Advisor"                               │
│       │   ├── image: quay.io/.../advisor-frontend@sha256:... │
│       │   ├── frontend.paths: ["/apps/advisor"]              │
│       │   ├── module:                                        │
│       │   │   ├── manifestLocation: /apps/advisor/fed-mods.  │
│       │   │   ├── modules: [{id, module, routes}]            │
│       │   │   └── analytics: {apiKey}                        │
│       │   ├── bundleSegments: [{bundleID, navItems}]         │
│       │   ├── searchEntries: [{title, href, description}]    │
│       │   ├── serviceTiles: [{section, group, title, href}]  │
│       │   └── widgetRegistry: [{scope, module, config}]      │
│       │                                                      │
│       └── Bundle (1 per navigation bundle)                   │
│           ├── id: "insights-navigation"                      │
│           ├── title: "Red Hat Insights"                      │
│           ├── envName: "stage"                               │
│           └── appList: ["advisor", "compliance", ...]        │
└─────────────────────────────────────────────────────────────┘
```

## Reconciliation — What the Operator Creates

For each `Frontend` CR, the operator creates or updates:

```
Frontend CR: "advisor"
        │
        ▼
┌─── Reconciliation Loop ──────────────────────────────────────┐
│                                                               │
│  1. Deployment: advisor-frontend                              │
│     ├── image: quay.io/.../advisor-frontend@sha256:...        │
│     ├── replicas: 1 (from FrontendEnvironment.defaultReplicas)│
│     ├── resources: 30m/50Mi requests, 40m/100Mi limits        │
│     └── volumes: ConfigMap mounts                             │
│                                                               │
│  2. Service: advisor                                          │
│     ├── port 8000 (HTTP)                                      │
│     └── port 9000 (metrics)                                   │
│                                                               │
│  3. Ingress: advisor                                          │
│     ├── ingressClass: openshift-default                       │
│     ├── rules: /apps/advisor/* → advisor:8000                 │
│     └── annotations: whitelist CIDRs, SSL termination         │
│                                                               │
│  4. ConfigMaps (environment-scoped, shared):                  │
│     ├── fed-modules.json      ← aggregated module registry    │
│     ├── bundles.json          ← assembled navigation          │
│     ├── search-index.json     ← merged search entries         │
│     ├── service-tiles.json    ← all services dropdown         │
│     ├── widget-registry.json  ← widget module metadata        │
│     ├── sso-config.json       ← SSO URL mappings              │
│     └── api-specs.json        ← OpenAPI spec locations        │
│                                                               │
│  5. ServiceMonitor: advisor (Prometheus scrape target)         │
│                                                               │
│  6. Cache Bust Job (conditional): advisor-frontend-cachebust  │
│                                                               │
│  7. Push Cache Job (conditional): advisor-frontend-pushcache  │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

## Configuration Aggregation

The operator aggregates data from **all** Frontend CRs into shared ConfigMaps:

```
Frontend: chrome          ──┐
Frontend: advisor         ──┤
Frontend: compliance      ──┤    ┌─────────────────────────────┐
Frontend: vulnerability   ──┼───▶│ ConfigMap: "stage"           │
Frontend: rbac            ──┤    │  fed-modules.json (all mods) │
Frontend: notifications   ──┤    │  bundles.json (all nav)      │
Frontend: landing         ──┤    │  search-index.json (all idx) │
Frontend: subscriptions   ──┤    │  service-tiles.json (tiles)  │
Frontend: ...30+ more     ──┘    └─────────────────────────────┘
                                          │
                                          ▼ distributed to
                                 ┌────────────────────┐
                                 │TargetNamespaces:    │
                                 │  ns-1/feo-context   │
                                 │  ns-2/feo-context   │
                                 │  ns-3/feo-context   │
                                 └────────────────────┘
```

## Key CRD Fields

### FrontendEnvironment

| Field | Type | Purpose |
|-------|------|---------|
| `hostname` | string | Environment hostname |
| `sso` / `ssoMapping` | string / map | SSO URL(s) for authentication |
| `bundles` | []FrontendBundles | Navigation bundle definitions |
| `serviceCategories` | []FrontendServiceCategory | All Services dropdown structure |
| `enableAkamaiCacheBust` | bool | Run Akamai cache invalidation jobs |
| `enablePushCache` | bool | Run valpop push-cache jobs |
| `targetNamespaces` | []string | Distribute ConfigMaps to these namespaces |
| `defaultReplicas` | *int32 | Default pod replica count |
| `reverseProxyImage` | string | Caddy reverse proxy image |
| `valpopImage` | string | Valpop cache populator image |

### Frontend

| Field | Type | Purpose |
|-------|------|---------|
| `envName` | string | Reference to FrontendEnvironment |
| `title` | string | Application display name |
| `image` | string | Container image (with digest) |
| `frontend.paths` | []string | URL paths served by this app |
| `module` | FedModule | Module federation config (manifest, modules, analytics) |
| `bundleSegments` | []BundleSegment | Navigation items per bundle |
| `searchEntries` | []SearchEntry | Search index entries |
| `serviceTiles` | []ServiceTile | All Services dropdown tiles |
| `widgetRegistry` | []WidgetModuleFederationMetadata | Widget definitions |
| `feoConfigEnabled` | bool | Include in operator-generated configs |

### Bundle

| Field | Type | Purpose |
|-------|------|---------|
| `id` | string | Bundle identifier (e.g., "insights-navigation") |
| `title` | string | Display name |
| `envName` | string | FrontendEnvironment reference |
| `appList` | []string | Application IDs in this bundle |
