(Written by Qwen)


### 🔍 Technical Review & Additions
Before drafting the consolidated document, here’s my assessment of the original discussion, along with critical additions for a production-ready tech lead guide:

**✅ What’s Strong:**
- Excellent separation of server vs. client state.
- Pragmatic UI strategy (headless vs. opinionated based on team capacity).
- Realistic offline/resilience approach (graceful degradation over full local-first).
- Clear anti-churn principles (thin wrappers, strict boundaries, web standards).

**⚠️ What’s Missing or Needs Adjustment:**
1. **Code Quality & Developer Experience Tooling:** ESLint, Prettier, Husky, and lint-staged are non-negotiable for team consistency but were omitted.
2. **Observability & Error Handling:** Production apps require Sentry (or similar) for runtime error tracking, React Error Boundaries, and basic telemetry. This was missing.
3. **Redux Dismissal:** While Zustand is ideal for 80% of cases, Redux Toolkit (RTK) still wins for apps requiring complex state machines, time-travel debugging, or enterprise teams with existing RTK expertise. I’ll position it as a valid fallback, not a dead end.
4. **React 19 / Next.js 15 Compatibility:** The stack should explicitly note compatibility with React 19’s compiler, `use()` hook, and Next.js 15 caching defaults, as these impact architecture choices today.
5. **Bundle & Performance Guardrails:** `@next/bundle-analyzer` or `rollup-plugin-visualizer` + CI budget checks prevent silent bloat. Tech leads need this.
6. **CI/CD Baseline:** While not a "library", a default stack implies a default pipeline. I’ll add a lightweight GitHub Actions + Vercel/Cloudflare deployment note.

---

# 📘 Default ReactJS Tech Stack: A Pragmatic Guide for Tech Leads
*Version 1.0 | Target Audience: Senior Engineers, Tech Leads, Architects*

## 🎯 Core Philosophy
> *"Boring technology wins."* This stack prioritizes **stability, modularity, talent availability, and clear architectural boundaries** over novelty. It is a starting point, not dogma. Every choice includes a fallback path based on team size, business constraints, and long-term maintainability.

---

## 🔀 1. The Strategic Fork: Meta-Framework Selection
Your first architectural decision dictates routing, rendering, and deployment strategy.

| Path | When to Choose | Key Trade-offs |
|------|----------------|----------------|
| **Vite + React Router (SPA)** | Internal tools, B2B SaaS behind auth, dashboards, SEO irrelevant, team prefers explicit routing | Faster dev build, simpler mental model, zero server-rendering complexity. No built-in SSR/SEO. |
| **Next.js 14/15 (App Router)** | Public-facing apps, B2C, e-commerce, SEO/OG critical, heavy server-side logic needed | Steeper learning curve (RSC, cache, `"use client"`), vendor-adjacent but industry-standard. |

**Decision Rule:** Let business requirements dictate this choice. If undecided, default to **Vite** to reduce cognitive overhead and avoid premature optimization.

---

## 📦 2. Core Default Stack

| Category | Default Choice | Rationale |
|----------|----------------|-----------|
| **Language** | TypeScript (Strict) | Catches runtime errors at compile time. Non-negotiable for maintainability. |
| **Package Manager** | `pnpm` | Strict dependency graph, disk-efficient, monorepo-ready. Faster CI/CD. |
| **Bundler / Dev Server** | `Vite` (SPA) or Next.js native | Native ES Modules, instant HMR, minimal config. |
| **Linting / Formatting** | `ESLint` + `Prettier` + `lint-staged` + `Husky` | Enforces consistency pre-commit. Prevents style debates in PRs. |
| **Client State** | `Zustand` | ~1KB, zero boilerplate, TypeScript-native. Scales cleanly for UI state. |
| **Server State / API** | `TanStack Query` + native `fetch` | Handles caching, retries, background refetch, loading/error states. Eliminates 80% of old Redux use cases. |
| **Validation** | `Zod` | Runtime validation that syncs with TS types. Use at API boundaries, form inputs, and prop validation. |
| **Forms** | `React Hook Form` + `@hookform/resolvers` (Zod) | Uncontrolled by default → vastly superior performance vs Formik. Native validation sync. |
| **UI Components** | `shadcn/ui` + `TailwindCSS` | Copy-paste, fully owned code. Avoids vendor lock-in. Pairs with Tailwind utility classes. |
| **Component Docs** | `Storybook` + `Chromatic` | Visual regression, designer/PM review, isolated component development. |
| **Testing** | `Vitest` + `@testing-library/react` + `Playwright` | Vitest shares Vite config. Playwright is fast, reliable, officially recommended by React team. Test behavior, not implementation. |

**Fallback Notes:**
- If TanStack Query is too hard to sell/onboard: **SWR** (simpler API, Vercel-backed).
- If complex state machines or extensive devtools are needed: **Redux Toolkit (RTK)**.
- If tables/data grids are heavy: `@tanstack/react-table` (headless, pairs with RHF/Zod).

---

## 🛠 3. Contextual Alternatives (Team & Resource Constraints)

| Scenario | Recommended Stack | Why |
|----------|-------------------|-----|
| **Under-resourced / Max Speed** | MUI or Ant Design + Theme Override | Pre-built, accessible, bulletproof. Escape customization pain by strictly using their design tokens. |
| **Intermediate / Balanced DX** | Mantine (`@mantine/core` + `@mantine/hooks`) | 100+ polished components, Tailwind/CSS-module friendly, built-in form/notification hooks. Reduces glue code. |
| **Enterprise / Legacy Teams** | RTK Query + Redux Toolkit | Familiar patterns, strong debugging ecosystem, predictable state transitions. Higher boilerplate but lower onboarding friction for veterans. |

---

## 🌐 4. Production Cross-Cutting Concerns

| Concern | Default Solution | Implementation Pattern |
|---------|------------------|------------------------|
| **Authentication** | Next.js: `Auth.js`<br>SPA: `Clerk` or `Supabase Auth` | **Never** store JWTs in `localStorage`. Use httpOnly cookies or managed SDKs. Delegate OAuth/MFA to proven providers. |
| **Real-Time Updates** | Native `WebSocket` / `EventSource` + TanStack Query | On message: `queryClient.setQueryData()` or `invalidateQueries()`. Keep UI components ignorant of transport layer. |
| **Offline Resilience** | `vite-plugin-pwa` + `@tanstack/query-persist-client` + `idb-keyval` | Precaches app shell. Serializes query cache to IndexedDB. Use optimistic UI + rollback for mutations. Avoid background sync queues unless explicitly required. |
| **Error Handling / Monitoring** | React Error Boundaries + `Sentry` | Catch render crashes client-side. Track unhandled promises, network failures, and performance regressions. |
| **Bundle & Perf Guardrails** | `@next/bundle-analyzer` or `rollup-plugin-visualizer` + CI size budgets | Fails PRs if critical chunk exceeds threshold. Prevents silent dependency bloat. |

---

## 🛡 5. Churn Mitigation & Future-Proofing Principles

The JavaScript ecosystem churns because teams tightly couple business logic to library internals. Apply these rules to survive 2025→2030:

1. **Strict Boundary Rules:** Never leak TanStack Query, Zustand, or Zod into UI components. Wrap them in custom hooks (`useGetUser`, `useAuth`, `useSubmitForm`). Swap the adapter, not the architecture.
2. **Prefer Web Standards over DSLs:** Avoid CSS-in-JS objects, styled-components, or bespoke query languages. Standard HTML/CSS/JS from 2015 works today; it will work in 2030.
3. **Modular over Monolithic:** The stack is intentionally flat. You can replace Zustand with Jotai, Vite with Turbopack, or Tailwind with vanilla CSS without touching your data or routing layer.
4. **Audit Dependencies Quarterly:** Use `npm audit`, `Snyk`, or `Dependabot`. Remove unused packages. Track bundle impact.

---

## 🧭 6. Quick Decision Flowchart

```
Start
  │
  ├─ SEO / SSR / Server Actions needed? ── Yes ──► Next.js (App Router)
  │                                       │
  │                                       No ──► Vite + React Router (SPA)
  │
  ├─ Team Size / UI Budget? ───────────── Small/Junior ──► MUI/Ant Design
  │                                       │
  │                                       Medium ──► shadcn/ui + Tailwind
  │                                       │
  │                                       Experienced ──► Mantine / Custom Headless
  │
  ├─ Real-time required? ──────────────── Yes ──► Native WS/SSE + TQ cache invalidation
  │
  ├─ Offline resilience required? ─────── Yes ──► PWA plugin + TQ IndexedDB persist + Optimistic UI
  │
  ├─ Auth required? ───────────────────── Yes ──► Clerk (SPA) / Auth.js (Next.js)
  │
  └─ Finalize: TS Strict + pnpm + ESLint/Prettier + Vitest/Playwright + Sentry
```

---

## 📝 Tech Lead Implementation Checklist
- [ ] Enforce `strict: true` in `tsconfig.json`
- [ ] Configure `pnpm` as default package manager + CI caching
- [ ] Set up pre-commit hooks (`lint-staged` + `husky`)
- [ ] Create `src/lib/` for data fetching, state, and auth wrappers
- [ ] Isolate UI components in `src/components/` (zero direct TQ/Zustand calls)
- [ ] Add Playwright E2E smoke tests to CI pipeline
- [ ] Integrate Sentry + define Error Boundary strategy
- [ ] Run bundle analyzer on first production build

---

### 💡 Final Note
This stack is designed to be **boring, modular, and talent-friendly**. It avoids paradigm shifts in favor of thin wrappers around web standards. As a tech lead, your primary responsibility isn’t picking immortal libraries—it’s enforcing boundaries that make swapping libraries trivial when the ecosystem inevitably moves on.

*Document Version:* 1.0 | *Last Updated:* 2024 | *Review Cadence:* Bi-annual or per major React/Next.js release.