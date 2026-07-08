# Platform Architecture Reference

Comprehensive architectural documentation for `console.redhat.com` — the Red Hat Hybrid Cloud Console.

These docs are split by domain so AI agents and developers can load only the context they need.

## Documents

| Document | When to read |
|----------|-------------|
| [Architecture Overview](architecture-overview.md) | High-level diagrams, request flow, infrastructure stack, key metrics |
| [insights-chrome (Shell)](insights-chrome.md) | Modifying the application shell, layouts, state management, component tree |
| [Frontend Operator](frontend-operator.md) | CRD schemas, reconciliation logic, config aggregation, deployment model |
| [Module Federation & Scalprum](module-federation.md) | App loading, shared singletons, chrome-service-backend, WebSocket |
| [Navigation & Auth](navigation-and-auth.md) | Navigation bundles, permission visibility, SSO flow, RBAC integration |
| [Theming](theming.md) | Theme layers, glass/felt/dark modes, state machine, logo variants |
| [Asset Pipeline](asset-pipeline.md) | Build → deploy → serve flow, valpop, Akamai cache busting |
| [Widgets & Shared Components](widgets-and-components.md) | Dashboard widget system, frontend-components monorepo (15 packages) |
| [AI Systems](ai-systems.md) | Chameleon chat widget, hcc-ai-assistant, Dev Bot, ai-web-clients |
| [Chrome API — useChrome()](chrome-api.md) | Complete API reference (~60 methods), backend services |
| [Environments & CI/CD](environments-and-cicd.md) | Environment matrix, CI/CD pipeline, shared workflows, repository map |

## For AI Agents

Add to your `CLAUDE.md`:

```markdown
## Platform Architecture Reference

Split docs in `frontend-experience-docs/pages/platform/`. Read on demand:
- Modifying chrome shell → read `pages/platform/insights-chrome.md`
- CRDs or operator work → read `pages/platform/frontend-operator.md`
- Navigation or auth/RBAC → read `pages/platform/navigation-and-auth.md`
- Theme changes → read `pages/platform/theming.md`
- AI/bot work → read `pages/platform/ai-systems.md`
- Build/deploy pipeline → read `pages/platform/asset-pipeline.md`
- useChrome() API → read `pages/platform/chrome-api.md`
- High-level context → read `pages/platform/architecture-overview.md`
```

---

*Maintainer: Console Framework Team (Platform Experience Services)*  
*Last updated: 2026-07-08*
