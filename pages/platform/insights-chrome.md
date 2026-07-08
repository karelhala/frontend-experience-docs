# insights-chrome — The Application Shell

**Repository:** [RedHatInsights/insights-chrome](https://github.com/RedHatInsights/insights-chrome)  
**Technology:** TypeScript, React 18, PatternFly 6, Webpack 5, Jotai  
**Role:** Host application for all micro-frontends. Provides authentication, navigation, theming, search, notifications, and the Chrome API to all consuming apps.

## Component Architecture

```
insights-chrome
├── bootstrap.tsx                    ← Entry point
├── components/
│   ├── RootApp/
│   │   ├── ScalprumRoot.tsx         ← Module federation host, route matching
│   │   └── ScalprumProvider         ← Provides Scalprum context to all apps
│   ├── Header/
│   │   ├── Header.tsx               ← Masthead layout (logo, search, toolbar)
│   │   ├── Logo.tsx                 ← Theme-aware logo (light/dark variants)
│   │   ├── Tools.tsx                ← Toolbar items (notifications, help, user, settings)
│   │   ├── UserToggle.tsx           ← User dropdown menu
│   │   ├── SettingsToggle.tsx       ← Settings gear button
│   │   └── SearchInput.tsx          ← Global search (Orama full-text engine)
│   ├── Navigation/
│   │   ├── Navigation.tsx           ← Sidebar navigation renderer
│   │   ├── ChromeNavItem.tsx        ← Individual nav item
│   │   └── DynamicNav.tsx           ← Dynamically loaded nav sections
│   ├── AllServices/
│   │   └── AllServicesDropdown.tsx   ← "All Services" mega-menu in masthead
│   ├── NotificationsDrawer/         ← Side drawer for live notifications
│   ├── GlobalFilter/                ← Global tag/filter bar (RHEL bundle)
│   ├── FeatureFlags/
│   │   └── FeatureFlagsProvider.tsx  ← Unleash proxy integration
│   ├── BetaSwitcher/                ← Preview/production toggle banner
│   └── Stratosphere/                ← Activation widget (trials, entitlements)
├── layouts/
│   ├── DefaultLayout.tsx            ← Standard layout: sidebar + header + content
│   └── Lightwell.tsx                ← Minimal layout: header + content only
├── chrome/
│   └── create-chrome.ts             ← Chrome API factory (~60 methods)
├── state/
│   ├── atoms/                       ← Jotai atoms (navigation, release, access, etc.)
│   └── stores/                      ← Scalprum shared stores (dark mode)
├── hooks/
│   ├── useGlassTheme.ts             ← Glass theme management
│   ├── useFeltTheme.ts              ← Felt theme activation
│   ├── useBundle.ts                 ← Bundle detection from URL
│   └── useSessionConfig.ts          ← User config persistence
├── utils/
│   ├── VisibilitySingleton.ts       ← Permission evaluation engine
│   ├── common.ts                    ← Environment detection, SSO routes
│   └── fetchNavigationFiles.ts      ← Navigation bundle fetcher
└── sass/
    └── chrome.scss                  ← Global styles, PF6 design tokens
```

## State Management

Chrome uses **Jotai** for atomic state and **Scalprum stores** for cross-app shared state:

```
Jotai Atoms (insights-chrome internal)
├── navigationAtom          ← Current navigation state, active items
├── activeModuleAtom        ← Currently loaded federated module
├── isPreviewAtom           ← User's preview/beta mode preference
├── scalprumConfigAtom      ← Module federation registry
├── gatewayErrorAtom        ← API gateway error state
├── contextSwitcherOpenAtom ← Account switcher visibility
├── layoutBannerHiddenAtom  ← Hide preview banner (layout override)
├── layoutForceGlassThemeAtom ← Force glass theme (layout override)
└── layoutLightwellHeaderAtom ← Simplified header mode (layout override)

Scalprum Shared Stores (cross-app via Module Federation)
└── ./theme/useDarkModeStore  ← Dark/light mode toggle (exposed to all apps)
```

## Layout System

```
┌─────────────────────────────────────────────────────────────┐
│ DefaultLayout                                                │
│ ┌─────────────────────────────────────────────────────────┐  │
│ │ Header (Masthead)                                        │  │
│ │ ┌──────┐ ┌──────────────────┐ ┌──────┐ ┌─────────────┐ │  │
│ │ │ Logo │ │AllServicesDropdown│ │Search│ │Toolbar      │ │  │
│ │ │      │ │                  │ │      │ │  N ? S U    │ │  │
│ │ └──────┘ └──────────────────┘ └──────┘ └─────────────┘ │  │
│ └─────────────────────────────────────────────────────────┘  │
│ ┌────────────┐ ┌──────────────────────────────────────────┐  │
│ │ Navigation │ │ Content Area                              │  │
│ │ (Sidebar)  │ │                                          │  │
│ │            │ │ ┌──────────────────────────────────────┐  │  │
│ │ > Overview │ │ │  GlobalFilter (optional)             │  │  │
│ │ > Advisor  │ │ └──────────────────────────────────────┘  │  │
│ │ > Vuln     │ │ ┌──────────────────────────────────────┐  │  │
│ │ > Compli.  │ │ │  Breadcrumbs                         │  │  │
│ │ > Patch    │ │ └──────────────────────────────────────┘  │  │
│ │ > Drift    │ │ ┌──────────────────────────────────────┐  │  │
│ │ > Policies │ │ │                                      │  │  │
│ │ > Inventory│ │ │  <ScalprumComponent />                │  │  │
│ │            │ │ │  (Federated Application)              │  │  │
│ │            │ │ │                                      │  │  │
│ │            │ │ └──────────────────────────────────────┘  │  │
│ └────────────┘ └──────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ Lightwell Layout (Minimal)                                   │
│ ┌─────────────────────────────────────────────────────────┐  │
│ │ Header (Simplified)                                      │  │
│ │ ┌──────┐ ┌───────────────────┐        ┌──────────────┐  │  │
│ │ │ Logo │ │"Red Hat Lightwell"│        │  User Menu   │  │  │
│ │ │(static)│                   │        │              │  │  │
│ │ └──────┘ └───────────────────┘        └──────────────┘  │  │
│ └─────────────────────────────────────────────────────────┘  │
│ ┌─────────────────────────────────────────────────────────┐  │
│ │                                                         │  │
│ │  <ScalprumComponent scope="contentSources"              │  │
│ │                      module="./LightwellApp" />         │  │
│ │                                                         │  │
│ │  (Full-bleed application, no sidebar, no breadcrumbs,   │  │
│ │   no search, no notifications, glass theme forced)      │  │
│ │                                                         │  │
│ └─────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Lightwell vs Default Comparison

| Feature | Default Layout | Lightwell Layout |
|---------|---------------|-----------------|
| Sidebar Navigation | Yes | No |
| GlobalFilter | Yes | No |
| Breadcrumbs | Yes (conditional) | No |
| Search Input | Yes | No |
| Logo Link | Yes (to `/`) | No (static) |
| Header Title | AllServicesDropdown | "Red Hat Lightwell" |
| Notifications/Help | Visible | Hidden |
| Preview Banner | Visible (toggleable) | Hidden (forced) |
| Glass Theme | User preference | Force-enabled |
| Virtual Assistant | Yes (if not ITLess) | No |
| Route Pattern | `/*` (default routes) | `/lightwell/*` |
