# Chrome API — useChrome()

Every federated application receives the Chrome API via the `useChrome()` hook from `@redhat-cloud-services/chrome`. This is the contract between the shell and its applications.

## API Reference

```
useChrome()
│
├── auth                              Authentication
│   ├── .getToken()                   Get current JWT access token
│   ├── .getRefreshToken()            Get refresh token
│   ├── .getUser()                    Get user identity + entitlements
│   ├── .login()                      Initiate login flow
│   ├── .logout()                     Logout user
│   ├── .doOffline()                  Enable offline mode
│   ├── .getOfflineToken()            Get offline token
│   └── .reAuthWithScopes(scopes)     Re-authenticate with specific scopes
│
├── Environment
│   ├── .isProd()                     Is production environment?
│   ├── .isBeta()                     Is preview/beta mode?
│   ├── .isDemo()                     Is demo mode?
│   ├── .isFedramp                    Is FedRAMP environment?
│   ├── .getEnvironment()             Get env string (prod/stage/qa/...)
│   ├── .getEnvironmentDetails()      Get detailed env info (URLs, SSO)
│   ├── .getBundle()                  Get current bundle name
│   ├── .getBundleData()              Get bundle ID + title
│   ├── .getApp()                     Get current app name
│   └── .getAvailableBundles()        List all bundles with titles
│
├── Navigation & Page
│   ├── .updateDocumentTitle(title)   Set browser tab title
│   ├── .identifyApp(appId, title)    Register current app
│   ├── .appAction(action)            Dispatch app action
│   ├── .appObjectId(id)              Set page object ID (telemetry)
│   ├── .appNavClick(item)            Notify nav item clicked
│   ├── .on(event, callback)          Listen: APP_NAVIGATION,
│   │                                   GLOBAL_FILTER_UPDATE,
│   │                                   NAVIGATION_TOGGLE
│   ├── .globalFilterScope(source)    Set global filter scope
│   ├── .hideGlobalFilter(hide)       Show/hide global filter
│   └── .chromeHistory                History object for routing
│
├── Permissions
│   ├── .getUserPermissions(app)      Get RBAC permissions (cached)
│   └── .visibilityFunctions          Full set of visibility checks
│       ├── .isOrgAdmin()
│       ├── .isActive()
│       ├── .isInternal()
│       ├── .isEntitled(bundle)
│       ├── .hasPermissions(perms)
│       ├── .loosePermissions(perms)
│       ├── .loosePermissionsKessel(...)
│       ├── .featureFlag(flag)
│       └── .apiRequest(url)
│
├── QuickStarts
│   ├── .quickStarts.set(items)       Set quickstart definitions
│   ├── .quickStarts.toggle(name)     Toggle quickstart drawer
│   ├── .quickStarts.Catalog          Catalog component
│   └── .quickStarts.activateQuickstart(name)  Open specific quickstart
│
├── Help Topics
│   ├── .helpTopics.addHelpTopics()   Register help topics
│   ├── .helpTopics.enableTopics()    Enable specific topics
│   ├── .helpTopics.disableTopics()   Disable topics
│   └── .helpTopics.setActiveTopic()  Set active topic
│
├── Analytics
│   ├── .analytics                    Segment AnalyticsBrowser instance
│   └── .segment.setPageMetadata()    Set page-level metadata
│
├── Search
│   ├── .search.query(term)           Query search results
│   └── .search.insert(entries)       Insert search entries
│
├── UI Controls
│   ├── .toggleDebuggerModal()        Toggle debugging modal
│   ├── .toggleFeedbackModal()        Toggle feedback modal
│   ├── .usePendoFeedback()           Trigger Pendo feedback
│   ├── .createCase()                 Create support case
│   ├── .requestPdf(component)        Generate PDF from React component
│   └── .drawerActions                Notification drawer panel controls
│       ├── .setDrawerPanelContent()
│       ├── .toggleDrawerPanel()
│       └── .toggleDrawerContent()
│
├── Module Federation
│   ├── .registerModule(mod, path)    Register federated module
│   └── .addWsEventListener(type, cb) Listen to WebSocket events
│
├── Debugging
│   └── .enable                       Debug enablers
│       ├── .iqe()                    IQE test mode
│       ├── .remediationsDebug()      Remediations debug
│       ├── .jwtDebug()               JWT debugging
│       ├── .reduxDebug()             Redux debugging
│       └── .quickstartsDebug()       Quickstarts debugging
│
└── Trials
    ├── .isAnsibleTrialFlagActive()   Check Ansible trial flag
    ├── .setAnsibleTrialFlag()        Set trial flag
    └── .clearAnsibleTrialFlag()      Clear trial flag
```

---

## Supporting Backend Services

```
┌─────────────────────────────────────────────────────────────┐
│  Core Platform Services                                      │
│                                                              │
│  chrome-service-backend (Go)                                 │
│  └── User prefs, nav config, WebSocket, search index         │
│                                                              │
│  quickstarts (Go)                                            │
│  └── Quickstart content storage & delivery                   │
│      Consumed via useChrome().quickStarts.activate()          │
│                                                              │
│  widget-layout-backend (Go + TS MCP sidecar)                 │
│  └── Dashboard template persistence, widget registry         │
│      API: /api/widget-layout/v1/                             │
│                                                              │
│  pdf-generator (TypeScript, Node.js)                         │
│  └── PDF reports via Puppeteer + headless Chrome             │
│      Renders federated React components as HTML → PDF        │
│                                                              │
│  dynamic-browser-events (Go, WebSocket)                      │
│  └── Real-time event push to connected browser clients       │
├─────────────────────────────────────────────────────────────┤
│  Infrastructure Services                                     │
│                                                              │
│  frontend-operator (Go)         K8s operator for CRDs        │
│  frontend-asset-proxy (Caddy)   Reverse proxy for S3 assets  │
│  valpop (Go)                    Asset cache (ValKey/Redis)    │
│  frontend-development-proxy     Local dev proxy               │
├─────────────────────────────────────────────────────────────┤
│  Code Generation                                             │
│                                                              │
│  javascript-clients (TypeScript)                             │
│  └── Auto-generated Axios clients from Swagger/OpenAPI       │
│                                                              │
│  insights-frontend-builder-common (Python, Shell)            │
│  └── Base images, build helpers, CI config for all apps      │
└─────────────────────────────────────────────────────────────┘
```
