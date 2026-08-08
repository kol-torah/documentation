
# Kol Torah — System Architecture

**Status:** Draft for review
**Last updated:** 2026-08-06

---

## 1. Goals and constraints

These are the requirements that drove every decision in this document. If one of
them changes, re-read the sections it touches.

| #  | Requirement                                                                      | Consequence                                              |
| -- | -------------------------------------------------------------------------------- | -------------------------------------------------------- |
| G1 | Public content must be indexable by search engines                               | Server-side rendering for public routes                  |
| G2 | Frontend logic and backend logic live in separate codebases with a hard boundary | SSR tier has no database access, ever                    |
| G3 | Some routes are for authenticated users only (admin, preferences)                | Session-aware rendering and authorization                |
| G4 | OIDC via the BFF pattern, supporting many IdPs                                   | Backend owns the OIDC flow; multi-provider config        |
| G5 | No hand-written authentication code                                              | Off-the-shelf OIDC package, no custom token handling     |
| G6 | Only the CDN caches; SSR and backend cache nothing                               | Explicit`Cache-Control` policy per route class         |
| G7 | Hot reload during development                                                    | Vite dev server + Django autoreload, source bind-mounted |
| G8 | Local environment closely resembles cloud deployment                             | Same containers, same routing topology, local edge proxy |
| G9 | No server management in the cloud                                                | Managed container runtime, managed Postgres              |

---

## 2. Stack

| Layer          | Choice                                                          | Notes                                                                                        |
| -------------- | --------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| Frontend + SSR | React Router v7/v8, framework mode                              | Formerly Remix; Vite-based, loaders/actions, no RSC requirement                              |
| Backend        | Django + django-ninja                                           | Django chosen for its auth ecosystem; django-ninja gives typed, Pydantic-based API endpoints |
| Authentication | `django-allauth`, `openid_connect` provider                 | Multiple IdPs configured as data, not code                                                   |
| Database       | Managed PostgreSQL                                              | Cloud SQL (GCP) or RDS (AWS)                                                                 |
| Session store  | Database-backed Django sessions                                 | State, not cache — does not violate G6                                                      |
| Runtime        | Containers on Cloud Run (GCP) or App Runner / ECS Fargate (AWS) | Scale to zero, no servers to patch                                                           |
| Edge           | Cloud CDN + external HTTPS LB, or CloudFront                    | Path-based routing, the only caching layer                                                   |

### Why React Router framework mode rather than Next.js

- The server boundary is explicit. A `loader` runs only on the server; the
  component runs on both sides. There is no "use client" ambiguity and no
  RSC mental model to teach.
- No framework caching layer to fight. Requirement G6 says the SSR tier caches
  nothing. React Router has no server cache by default and lets each route
  declare its own response headers, which is exactly the shape we want.
- Deploys as an ordinary Node process in an ordinary container. No adapters,
  no platform-specific build output.
- Per-route type generation gives typed loader data and params without manual
  annotation.

Accepted trade-offs: smaller ecosystem than Next.js, no built-in ISR or image
optimizer. Neither matters here — content freshness is driven by CDN purges from
the backend, and images go through the CDN or a dedicated service.

---

## 3. Component overview

```
                       ┌──────────────┐
                       │   Browser    │
                       └──────┬───────┘
                              │  HTTPS, session cookie
                       ┌──────▼───────────────┐
                       │  CDN + edge router   │   ← the only cache in the system
                       └──┬────────────────┬──┘
              page routes │                │ /api/*, /accounts/*
                       ┌──▼────────────┐   │
                       │  SSR server   │   │
                       │  React Router │   │
                       │  no db        │   │
                       │  no cache     │   │
                       │  no auth code │   │
                       └──┬────────────┘   │
                          │ forwards cookie│
                       ┌──▼────────────────▼──┐
                       │   Django backend     │
                       │   sessions, API,     │
                       │   OIDC (BFF)         │
                       └──────┬───────────────┘
                              │
                    ┌─────────▼──────────┐
                    │     PostgreSQL     │
                    └────────────────────┘
```

### Tier responsibilities

**Edge router / CDN**

- Terminates TLS, routes by path prefix.
- Caches responses that carry public cache headers. Honours origin headers
  rather than applying a blanket TTL.
- Accepts purge requests from the backend.

**SSR server (React Router)**

- Renders HTML for public content and the shell for authenticated routes.
- Calls the backend over HTTP for all data.
- Forwards the incoming `Cookie` header to the backend on every server-side
  fetch.
- Holds no database credentials, no IdP credentials, no session state, and no
  cache.

**Backend (Django)**

- Owns the database and all domain logic.
- Owns the entire OIDC flow: it is the BFF.
- Issues and validates session cookies.
- Enforces authorization on every endpoint, independent of what the frontend
  chose to render.
- Purges CDN paths when content changes.

---

## 4. Routing

A single origin serves everything, so cookies flow without CORS and without
`SameSite=None`.

| Path prefix        | Target                                    | Cacheable              |
| ------------------ | ----------------------------------------- | ---------------------- |
| `/api/*`         | Django                                    | No                     |
| `/accounts/*`    | Django (allauth: login, callback, logout) | No                     |
| `/admin/*`       | SSR (client-rendered shell)               | No                     |
| `/preferences/*` | SSR                                       | No                     |
| everything else    | SSR                                       | Yes, per route headers |

The same routing table is expressed three times, and all three must agree:

1. Local development — Caddy or nginx container.
2. Staging and production — load balancer / CloudFront behaviour config.
3. React Router's `routes.ts`, which must not claim any path owned by Django.

Keep the table in one place in the repo (a small JSON or YAML file) and generate
the Caddyfile and the Terraform config from it, so the three cannot drift apart.

---

## 5. Authentication

### The BFF pattern, with Django as the BFF

The requirement to write no authentication code (G5) decides which tier is the
BFF. The backend already needs an OIDC package for session management, so it
performs the flow; the SSR tier never sees a token.

**Login**

1. User clicks a provider on the login screen; browser navigates to
   `/accounts/oidc/<provider_id>/login/`.
2. The edge routes it to Django. `django-allauth` performs the authorization
   code flow with PKCE against that provider.
3. The IdP redirects to
   `/accounts/oidc/<provider_id>/login/callback/`.
4. Django exchanges the code, resolves or creates the local `User`, and sets a
   session cookie: `HttpOnly`, `Secure`, `SameSite=Lax`, apex domain.
5. Tokens are stored server-side. They never reach the browser or the SSR tier.

**Authenticated page render**

1. Browser requests `/admin/lessons`. The cookie is attached automatically.
2. The edge routes it to SSR. The response is not cached.
3. The route's `loader` copies the `Cookie` header onto its fetch to
   `/api/...`.
4. Django resolves cookie → session → user, authorizes the request, and returns
   data or a 401/403.
5. SSR renders. On 401 it issues a redirect to the login screen.

**Client-side navigation after hydration**

Fetches go to `/api/...` on the same origin. The cookie is attached by the
browser; no forwarding needed.

### Cookie forwarding

Every loader that needs identity goes through one helper. Nothing else may call
`fetch` against the backend.

```ts
// app/lib/api.server.ts
const API = process.env.INTERNAL_API_URL!;

export async function apiFetch(
  request: Request,
  path: string,
  init: RequestInit = {},
) {
  const res = await fetch(`${API}${path}`, {
    ...init,
    headers: {
      ...init.headers,
      cookie: request.headers.get("cookie") ?? "",
      "x-forwarded-for": request.headers.get("x-forwarded-for") ?? "",
    },
  });
  if (res.status === 401) throw redirect("/login");
  return res;
}
```

A lint rule should ban bare `fetch` to the API host outside this module. This is
the single point where requirement G2's identity propagation is implemented, and
it should stay that way.

### Multiple identity providers

`django-allauth`'s `openid_connect` provider registers several independent OIDC
sub-providers, each as an entry in `APPS`, each with its own `provider_id` and
its own callback URL. Adding an IdP is a settings change and a client
registration — no code.

```python
SOCIALACCOUNT_PROVIDERS = {
    "openid_connect": {
        "OAUTH_PKCE_ENABLED": True,
        "APPS": [
            {
                "provider_id": "google",
                "name": "Google",
                "client_id": env("GOOGLE_CLIENT_ID"),
                "secret": env("GOOGLE_CLIENT_SECRET"),
                "settings": {"server_url": "https://accounts.google.com"},
            },
            {
                "provider_id": "local-keycloak",
                "name": "Development",
                "client_id": env("KC_CLIENT_ID"),
                "secret": env("KC_CLIENT_SECRET"),
                "settings": {"server_url": env("KC_ISSUER")},
            },
        ],
    }
}
```

Build a real provider-chooser login screen from day one rather than a single
"Sign in" button — the multi-IdP requirement is structural, not a later feature.

Note that providers differ on RP-initiated logout: `end_session_endpoint`
appears in the discovery document only when the provider supports that flow.
Logout must handle both cases — always clear the local session, and redirect to
the IdP's end-session endpoint only when one is published.

### Authorization

Authorization is a backend concern. The SSR tier decides what to *render*; the
backend decides what to *return*. Every endpoint checks permissions on its own,
so a bug in a frontend route guard cannot expose data.

---

## 6. Caching

Only the CDN caches. This is the highest-risk area in the design: caching a page
that was rendered for a signed-in user leaks their data to everyone.

### Route classes

**Public, cacheable.** Lessons, rabbis, search pages, static content. These
render identically for every visitor. No user-specific data appears in the SSR
output at all — no name in the header, no admin link. User chrome is fetched
client-side from `/api/me` after hydration.

```ts
export function headers() {
  return {
    "Cache-Control": "public, s-maxage=300, stale-while-revalidate=86400",
  };
}
```

**Private, never cached.** `/admin/*`, `/preferences/*`, `/api/*`,
`/accounts/*`.

```ts
export function headers() {
  return {
    "Cache-Control": "private, no-store, max-age=0, must-revalidate",
  };
}
```

### Rules

- No route may render both public and user-specific content. If a page needs
  both, the user-specific part is a client-side fetch.
- Add a CI check that fails if any route module omits a `headers` export.
  Silence is the dangerous default here.
- Configure the CDN to respect origin cache headers. Do not set a default TTL
  at the edge.
- The backend purges CDN paths after content edits. Publishing a lesson should
  invalidate the lesson page, the index, and any listing that includes it.

---

## 7. Local development

`docker compose` with five services, all reached through a single origin at
`http://localhost:8080`.

| Service  | Contents                                    | Hot reload                |
| -------- | ------------------------------------------- | ------------------------- |
| `edge` | Caddy, path routing identical to production | Config reload on change   |
| `web`  | React Router dev server (Vite)              | HMR via bind mount        |
| `api`  | Django`runserver`                         | Autoreload via bind mount |
| `db`   | PostgreSQL                                  | —                        |
| `idp`  | Keycloak with a pre-seeded realm            | —                        |

The `edge` service is what makes G8 real. Routing is the thing most likely to
surprise you at deploy time, so it must be exercised on every developer machine.

Dockerfiles are multi-stage with a `dev` target and a `prod` target, so the
production image is built from the same file the developers use.

### Why Keycloak rather than a real IdP for daily work

- Test users can be created, mutated, and deleted freely; claims can be
  removed or renamed to check how the backend behaves.
- Token lifetimes can be set to seconds, so refresh and expiry paths are
  actually exercised.
- End-to-end tests can log in headlessly. Scripted logins against a real
  consumer IdP break on bot detection and 2FA.
- New contributors get a working login from `docker compose up`, with no real
  client secrets distributed.

Real providers are still validated, on staging, before launch — see §8.

---

## 8. Deployment

### GCP (preferred)

- Two Cloud Run services: `web` (SSR) and `api` (Django), same region.
- External HTTPS load balancer with two serverless NEG backends and the
  path-matching rules from §4.
- Cloud CDN enabled on the public backend only.
- Cloud SQL for PostgreSQL, private IP.
- Secret Manager for IdP client secrets and the Django secret key.

### AWS

- Two App Runner services, or ECS Fargate services behind an ALB.
- CloudFront with two origins and path-based behaviours.
- RDS PostgreSQL.
- Secrets Manager for IdP client secrets and the Django secret key.

### Network security

The backend must not be reachable from the internet except through the load
balancer paths chosen in §4. On Cloud Run, make `api` require IAM
authentication and have `web` attach a signed ID token to its calls; on AWS,
place the backend in a private subnet.

Use a keep-alive HTTP agent for the SSR-to-backend hop. Same-region container
to container is a millisecond or two, which is why the strict separation in G2
is affordable.

### Environments

| Environment | Purpose                                                                       |
| ----------- | ----------------------------------------------------------------------------- |
| local       | Keycloak, seeded data, full stack in compose                                  |
| staging     | Real Google plus one enterprise IdP (Entra ID), real CDN, production topology |
| production  | —                                                                            |

Staging exists specifically to validate the IdP configuration and CDN cache
behaviour, which local Keycloak and Caddy cannot fully rehearse.

---

## 9. Boundary rules

These are the invariants that keep G2 intact. Enforce them in CI, not by memory.

1. The SSR service has no database driver in its dependency tree and no
   database credentials in its environment.
2. The SSR service holds no IdP client secrets.
3. All backend calls from SSR go through `apiFetch`. Bare `fetch` to the API
   host fails lint.
4. Every React Router route module exports `headers`.
5. Resource routes in the SSR app are limited to a documented allowlist —
   health checks and similar. They are not a place for business logic.
6. Authorization is checked in the backend for every endpoint, regardless of
   frontend route guards.

---

## 10. Open decisions

- **Admin rendering mode.** Recommendation: render `/admin/*` client-side
  (`ssr: false` for that route branch) within the same app. It is behind login,
  never indexed, never cached, and this removes cookie forwarding from the most
  privilege-sensitive screens. Confirm before building.
- **Search.** Postgres full-text search is likely sufficient at first and avoids
  a component. Revisit if Hebrew stemming and nikud handling prove inadequate;
  the alternative is a managed OpenSearch or Typesense instance.
- **Content model.** Not covered here. The relationship between rabbis,
  lessons, and series needs its own document.
- **CDN purge granularity.** Per-path purge versus surrogate keys. Cloud CDN's
  invalidation is coarse and rate-limited; if content edits are frequent, a
  fronting CDN with tag-based purge may be worth the added component.
- **Image handling.** Deferred until the content model is settled.
- **Observability.** Structured logging, trace propagation from edge through SSR
  to backend. The `traceparent` header should be forwarded by `apiFetch`
  alongside the cookie.
