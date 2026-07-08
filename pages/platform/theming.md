# Theming & Visual System

## Theme Layers

```
┌─────────────────────────────────────────────────────────────┐
│ Layer 1: PatternFly 6 Design Tokens (CSS Custom Properties)  │
│                                                              │
│  --pf-t--global--color--brand--default                       │
│  --pf-t--global--text--color--regular                        │
│  --pf-t--global--spacer--md                                  │
│  --pf-t--global--font--size--body--default                   │
│  ... 500+ design tokens                                      │
└──────────────────────────────┬──────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────┐
│ Layer 2: Theme Classes (document.documentElement)            │
│                                                              │
│  Default (light)  → no class                                 │
│  Dark mode        → .pf-v6-theme-dark                        │
│  Glass mode       → .pf-v6-theme-glass                       │
│  Felt mode        → .pf-v6-theme-felt                        │
└──────────────────────────────┬──────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────┐
│ Layer 3: Chrome Overrides (chrome.scss)                       │
│                                                              │
│  .chr-c-masthead { ... }                                     │
│  .chr-c-navigation { ... }                                   │
│  .chr-c-page { ... }                                         │
│  Dark mode link colors, layout adjustments                   │
└──────────────────────────────┬──────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────┐
│ Layer 4: App-Specific Styles (per federated module)          │
│                                                              │
│  Each app brings its own CSS, scoped to content area         │
│  Must use PF design tokens for theme consistency             │
└──────────────────────────────────────────────────────────────┘
```

## Theme State Machine

```
                    ┌─────────────┐
                    │  App Load   │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │ Check glass │
                    │ feature flag│
                    └──────┬──────┘
                           │
                ┌──────────┼──────────┐
                │ enabled  │          │ disabled
                ▼          │          ▼
         ┌────────────┐    │   ┌────────────┐
         │Force glass?│    │   │ Check dark │
         │(layout atom)│   │   │mode store  │
         └─────┬──────┘    │   └─────┬──────┘
               │           │         │
          ┌────┼────┐      │    ┌────┼────┐
          │yes │    │no    │    │yes │    │no
          ▼    │    ▼      │    ▼    │    ▼
     ┌────────┐│┌────────┐ │┌───────┐│┌───────┐
     │ Glass  │││ Read   │ ││ Dark  │││ Light │
     │ Theme  │││ user   │ ││ Theme │││ Theme │
     │(forced)│││ pref   │ ││       │││       │
     └────────┘│└────────┘ │└───────┘│└───────┘
               │           │         │
               │    localStorage     │
               │    "glass_theme"    │
               │           │         │
               └───────────┘         │
                                     │
                    Scalprum store:   │
                    useDarkModeStore ─┘
                    (shared across all apps)
```

## Logo Variants

```
Light theme  →  /static/images/logo.svg      (Red Hat logo, standard)
Dark theme   →  /static/images/logo-dark.svg  (Red Hat logo, inverted)
```

## Key Files

| File | Purpose |
|------|---------|
| `src/state/stores/darkModeStore.ts` | Scalprum shared store for dark/light toggle |
| `src/hooks/useGlassTheme.ts` | Glass theme management with force-enable support |
| `src/hooks/useFeltTheme.ts` | Felt theme class activation |
| `src/state/atoms/releaseAtom.ts` | Layout override atoms (force glass, hide banner) |
| `src/sass/chrome.scss` | Global styles, PF6 design tokens, dark mode overrides |
| `src/components/Header/Logo.tsx` | Theme-aware logo component |
