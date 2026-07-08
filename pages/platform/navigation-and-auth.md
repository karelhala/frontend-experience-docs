# Navigation & Authentication

## Navigation Bundles

Navigation is organized into **bundles**, each representing a product area:

```
┌────────────────────────────────────────────────────────────────┐
│                    Navigation Bundles                           │
│                                                                │
│  insights          "Red Hat Insights"      (RHEL)              │
│  openshift         "OpenShift"             (OpenShift)         │
│  ansible           "Ansible"               (AAP)               │
│  application-services  "App Services"      (Streams, etc.)     │
│  settings          "Settings"              (Integrations, etc.)│
│  iam               "IAM"                   (RBAC, Users)       │
│  landing           "Landing"               (Home page)         │
│  quay              "Quay.io"               (Container registry)│
│  subscriptions     "Subscriptions"         (Subs dashboard)    │
│  docs              "Documentation"         (Docs)              │
│  internal          "Internal"              (RH-only features)  │
│  user-preferences  "User Preferences"      (User settings)     │
└────────────────────────────────────────────────────────────────┘
```

## Navigation Assembly Pipeline

```
Frontend CRD:advisor           Frontend CRD:compliance
  bundleSegments:                bundleSegments:
  - bundleID: insights           - bundleID: insights
    position: 200                  position: 300
    navItems:                      navItems:
    - title: "Advisor"             - title: "Compliance"
      href: /insights/advisor        href: /insights/compliance
                │                          │
                └────────┬─────────────────┘
                         │
                         ▼
          ┌──── Operator Aggregation ────┐
          │                              │
          │  1. Collect all segments     │
          │     for bundle "insights"     │
          │                              │
          │  2. Sort by position          │
          │     200 → Advisor             │
          │     300 → Compliance          │
          │     400 → Vulnerability       │
          │     ...                       │
          │                              │
          │  3. Resolve SegmentRefs       │
          │     (cross-frontend refs)     │
          │                              │
          │  4. Write bundles.json        │
          └──────────────────────────────┘
                         │
                         ▼
          ┌──── Chrome Runtime ──────────┐
          │                              │
          │  Fetch bundles-generated.json │
          │  Match URL to bundle         │
          │  Render sidebar NavItems      │
          │  Apply permission filtering   │
          │  Highlight active item        │
          └──────────────────────────────┘
```

## Permission-Based Visibility

Each navigation item can declare visibility rules evaluated at render time:

```
Visibility Functions (VisibilitySingleton)
├── isOrgAdmin       ← User is org administrator
├── isActive         ← Account is active
├── isInternal       ← @redhat.com email domain
├── isEntitled       ← Has specific entitlement
├── hasPermissions   ← RBAC permission check (AND logic)
├── loosePermissions ← RBAC permission check (OR logic)
├── loosePermissionsKessel ← Kessel-native tenant-scoped permissions
├── featureFlag      ← Unleash feature flag enabled
├── apiRequest       ← Custom API endpoint returns truthy
├── withEmail        ← Email domain match
├── hasCookie        ← Specific cookie exists
├── hasLocalStorage  ← Specific localStorage key exists
├── isProd           ← Production environment only
└── isBeta           ← Preview/beta mode only
```

---

## Authentication — SSO Flow

```
┌──────────┐         ┌───────────────┐         ┌───────────────┐
│  Browser │────1───▶│ insights-     │────2───▶│  Keycloak     │
│          │         │ chrome        │         │  (SSO)        │
│          │◀───5────│               │◀───3────│               │
│          │         │               │         │               │
│          │────6───▶│ useChrome()   │         │               │
│          │         │ .auth.getToken│         │               │
└──────────┘         └───────────────┘         └───────────────┘
                            │
                     4. Token stored
                        in cookie

1. User navigates to console.redhat.com
2. Chrome redirects to SSO login (Keycloak)
3. SSO returns JWT access token + refresh token
4. Token stored in cookie, refresh timer started
5. Shell renders with authenticated context
6. Apps call useChrome().auth.getToken() for API calls
```

## Environment-Specific SSO

```
Environment          SSO Endpoint
─────────────────────────────────────────────────────────
Production           sso.redhat.com
Stage                sso.stage.redhat.com
QA                   sso.qa.redhat.com
CI                   sso.qa.redhat.com
FRH (GovCloud)       sso.stage.openshiftusgov.com (stage)
                     sso.openshiftusgov.com (prod)
Ephemeral            sso.stage.redhat.com
```

## RBAC Integration

```
App requests permissions:

useChrome().getUserPermissions("advisor")
        │
        ▼
  RBAC API call: GET /api/rbac/v1/access/?application=advisor
        │
        ▼
  Returns: [
    { permission: "advisor:*:*",     resourceDefinitions: [] },
    { permission: "advisor:topic:read", resourceDefinitions: [...] }
  ]
        │
        ▼
  Cached in VisibilitySingleton for subsequent checks
```
