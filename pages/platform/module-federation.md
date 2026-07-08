# Module Federation & Chrome Service Backend

## How Apps Are Loaded at Runtime

```
                     insights-chrome (HOST)
                     Webpack ModuleFederationPlugin
                     name: "chrome"
                            │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
        ┌──────────┐  ┌──────────┐  ┌──────────┐
        │ advisor  │  │compliance│  │  rbac    │
        │ REMOTE   │  │ REMOTE   │  │ REMOTE   │  ... 30+ remotes
        │          │  │          │  │          │
        │fed-mods  │  │fed-mods  │  │fed-mods  │
        │  .json   │  │  .json   │  │  .json   │
        └──────────┘  └──────────┘  └──────────┘

Shared singletons (loaded once, used by all):
├── react (18.3.x)
├── react-dom (18.3.x)
├── react-router-dom (6.x)
├── @patternfly/react-core (6.x)
├── @patternfly/react-icons (6.x)
├── @patternfly/react-table (6.x)
├── @scalprum/core
├── @scalprum/react-core
├── @unleash/proxy-client-react
├── @redhat-cloud-services/frontend-components
├── @redhat-cloud-services/frontend-components-utilities
├── @redhat-cloud-services/frontend-components-notifications
└── jotai
```

## Chrome Exposed Modules

Other apps can import these from chrome at runtime:

```javascript
"./LandingNavFavorites"           // Favorite services component
"./DashboardFavorites"            // Dashboard favorites widget
"./SatelliteToken"                // Satellite token auth layout
"./ModularInventory"              // Inventory POC component
"./search/useSearch"              // Search hook
"./analytics/intercom/*"          // Intercom integration
"./theme/useDarkModeStore"        // Dark mode toggle (Scalprum store)
```

## Scalprum Loading Sequence

```
1. ScalprumProvider initializes with module config
           │
2. URL pathname matched against route registry
           │
3. ScalprumComponent resolves scope + module
           │
4. Webpack runtime loads remote entry JS
           │
5. Shared dependencies resolved (singletons)
           │
6. Remote module factory executed
           │
7. React component rendered in content area
           │
8. App receives ChromeAPI via useChrome() hook
```

## Error Recovery

```
Module load fails
    │
    ├── ChunkLoadError? → 10-second countdown → page refresh
    │
    ├── Network error?  → lazyWithRetry wrapper retries import
    │
    └── Unknown error?  → ScalprumError boundary → error page
```

---

## Chrome Service Backend

**Repository:** [RedHatInsights/chrome-service-backend](https://github.com/RedHatInsights/chrome-service-backend)  
**Technology:** Go  
**Role:** Backend service supporting the shell — user preferences, generated configs, search, WebSocket notifications.

### API Endpoints

```
/api/chrome-service/v1/
├── static/
│   ├── bundles-generated.json          ← Navigation bundles (from operator)
│   ├── fed-modules-generated.json      ← Module registry (from operator)
│   ├── search-index-generated.json     ← Search entries (from operator)
│   ├── service-tiles-generated.json    ← All Services tiles (from operator)
│   ├── sso-config-generated.json       ← SSO URL mapping
│   └── {env}/{bundle}-navigation.json  ← Legacy per-bundle nav files
├── user/
│   ├── GET  /                          ← Fetch user config
│   ├── POST /update-ui-preview         ← Toggle preview mode
│   ├── POST /mark-preview-seen         ← Mark preview banner seen
│   └── GET  /intercom                  ← Intercom user hash
└── ws/
    └── wss://*/wss/chrome-service/v1/ws ← WebSocket (CloudEvents JSON)
                                            Event: com.redhat.console.notifications.drawer
```

### WebSocket Architecture

```
┌──────────┐    WebSocket     ┌────────────────┐    CloudEvents    ┌──────────────┐
│  Browser │◄────────────────▶│ chrome-service  │◄─────────────────│ Notification │
│ (chrome) │  Auto-reconnect  │    backend      │                  │   Service    │
│          │  5 retries, 2s   │                 │                  │              │
└──────────┘  backoff         └────────────────┘                  └──────────────┘

Event format (CloudEvents JSON):
{
  "specversion": "1.0",
  "type": "com.redhat.console.notifications.drawer",
  "source": "/notifications",
  "data": { ... notification payload ... }
}
```
