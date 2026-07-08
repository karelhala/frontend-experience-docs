# Asset Pipeline — Build, Cache, Serve

```
┌──── Build Phase ──────────────────────────────────────────────────┐
│                                                                    │
│  GitHub PR merge                                                   │
│       │                                                            │
│       ▼                                                            │
│  Konflux / GitHub Actions                                          │
│       │                                                            │
│       ├── npm install → webpack build → content-hashed chunks      │
│       │   chrome-root.[abc123].js                                  │
│       │   chunk.[def456].js                                        │
│       │   fed-mods.json (manifest)                                 │
│       │                                                            │
│       ├── Container image built (Dockerfile)                       │
│       │   → quay.io/.../{app}-frontend@sha256:...                  │
│       │                                                            │
│       └── Image pushed to Quay registry                            │
│                                                                    │
└───────────────────────────────┬────────────────────────────────────┘
                                │
┌──── Deploy Phase ─────────────▼───────────────────────────────────┐
│                                                                    │
│  Frontend CRD updated with new image digest                        │
│       │                                                            │
│       ▼                                                            │
│  frontend-operator reconciles                                      │
│       │                                                            │
│       ├── Updates Deployment with new image                        │
│       ├── Runs Push Cache Job (valpop)                              │
│       │   └── Dumps assets into ValKey/Redis cache                 │
│       │       (enables rollback to previous versions)              │
│       ├── Runs Cache Bust Job                                      │
│       │   └── Invalidates Akamai edge cache for app paths          │
│       └── Updates ConfigMaps (fed-modules, nav, search, etc.)      │
│                                                                    │
└───────────────────────────────┬────────────────────────────────────┘
                                │
┌──── Serve Phase ──────────────▼───────────────────────────────────┐
│                                                                    │
│  Browser request: /apps/advisor/chunk.[hash].js                    │
│       │                                                            │
│       ▼                                                            │
│  Akamai CDN edge                                                   │
│       │                                                            │
│       ├── Cache HIT → serve from edge (< 50ms)                     │
│       │                                                            │
│       └── Cache MISS                                               │
│            │                                                       │
│            ▼                                                       │
│       frontend-asset-proxy (Caddy)                                 │
│            │                                                       │
│            ▼                                                       │
│       S3 / MinIO object storage                                    │
│       (current assets served directly)                             │
│                                                                    │
│  Rollback scenario:                                                │
│       valpop retrieves previous version from ValKey/Redis cache    │
│       → restores assets to S3 → Akamai cache busted               │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

## Key Services

| Service | Technology | Purpose |
|---------|-----------|---------|
| **frontend-asset-proxy** | Go, Caddy | Reverse proxy serving assets from S3/MinIO |
| **valpop** | Go | Asset versioning in ValKey/Redis cache, rollback support |
| **Akamai** | CDN | Edge caching, cache busting via operator jobs |
| **Konflux** | CI/CD | Container image builds, security scans, signing |
| **App-Interface** | GitOps | Image digest updates, deployment promotion |
