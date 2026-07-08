# Widget Dashboard & Shared Components

## Widget Dashboard System

**Repositories:**  
- [widget-layout](https://github.com/RedHatInsights/widget-layout) (Frontend)  
- [widget-layout-backend](https://github.com/RedHatInsights/widget-layout-backend) (Backend)

```
┌─────────────────────────────────────────────────────────────────┐
│  Dashboard (Landing Page)                                        │
│                                                                  │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐            │
│  │ RHEL Widget  │ │ ACS Widget   │ │ Support Case │            │
│  │              │ │              │ │    Widget    │            │
│  │ (landing     │ │ (landing     │ │ (landing     │            │
│  │  ./Rhel)     │ │  ./Acs)      │ │  ./Support)  │            │
│  ├──────────────┤ ├──────────────┤ ├──────────────┤            │
│  │  drag        │ │  drag        │ │  drag        │            │
│  └──────────────┘ └──────────────┘ └──────────────┘            │
│  ┌──────────────┐ ┌──────────────┐                              │
│  │ OpenShift    │ │ Explore      │   + Add Widget               │
│  │   Widget     │ │   Widget     │     (Widget Drawer)          │
│  │              │ │              │                              │
│  └──────────────┘ └──────────────┘                              │
│                                                                  │
│  Grid: react-grid-layout (responsive)                            │
│  Breakpoints: sm(1col) md(2col) lg(3col) xl(4col)               │
│  Persistence: /api/widget-layout/v1/ (debounced 1500ms)          │
└─────────────────────────────────────────────────────────────────┘
```

### Widget Loading Flow

```
1. Frontend fetches widget registry
   GET /api/widget-layout/v1/widget-mapping
        │
        ▼
2. Registry maps widgetType → Scalprum config
   {
     "rhel-widget": {
       scope: "landing",
       module: "./RhelWidget",
       defaults: { w: 1, h: 2, maxH: 4, minH: 1 },
       config: {
         title: "RHEL",
         icon: "RhelIcon",
         permissions: [{ method: "isEntitled", args: ["insights"] }]
       }
     }
   }
        │
        ▼
3. Permission filtering (client-side)
   Remove widgets user lacks permission for
        │
        ▼
4. Scalprum loads each widget as federated module
   Same mechanism as full apps, but rendered in grid cells
```

---

## frontend-components Monorepo

**Repository:** [RedHatInsights/frontend-components](https://github.com/RedHatInsights/frontend-components)  
**Build System:** Nx 22.x with independent versioning  
**Release:** Conventional commits → automated npm publish with provenance

```
@redhat-cloud-services/
├── frontend-components                  Core UI components
├── frontend-components-utilities        Utility functions & helpers
├── chrome                               Console integration (useChrome wrapper)
├── frontend-components-notifications    Toast notification portal
├── frontend-components-remediations     Remediation wizard
├── frontend-components-advisor-components  Advisor domain components
├── rule-components                      Rule information display
├── types                                Shared TypeScript definitions
├── frontend-components-config           Build/runtime config plugins
├── frontend-components-config-utilities Config utility helpers
├── frontend-components-testing          Test utilities
├── frontend-components-translations     i18n utilities
├── eslint-config-redhat-cloud-services  Shared ESLint config
├── tsc-transform-imports                TS compiler import optimizer
└── frontend-components-executors        Nx executors (tooling)
```

### Dependency Graph

```
┌─────────────────────────────────────────────────────┐
│              Consuming Application                   │
│  (e.g., advisor-frontend, compliance-frontend)       │
│                                                      │
│  import Ansible from                                 │
│    '@redhat-cloud-services/frontend-components/      │
│     Ansible';                                        │
│                                                      │
│  import { useChrome } from                           │
│    '@redhat-cloud-services/chrome';                  │
│                                                      │
│  import { addNotification } from                     │
│    '@redhat-cloud-services/frontend-components-      │
│     notifications';                                  │
└──────────┬──────────────────────────────┬────────────┘
           │                              │
           ▼                              ▼
┌──────────────────┐          ┌──────────────────────┐
│ frontend-        │          │ insights-chrome      │
│ components (npm) │          │ (Module Federation)  │
│                  │          │                      │
│ Build-time dep   │          │ Runtime shared       │
│ (tree-shaken)    │          │ singleton            │
└──────────────────┘          └──────────────────────┘
```
