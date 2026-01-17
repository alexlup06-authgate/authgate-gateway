# AuthGate Gateway

**AuthGate Gateway** is a lightweight, configuration-driven reverse proxy that
routes traffic between your application and **AuthGate**.

Built on **Caddy**, it is fully environment-agnostic and behaves identically
across all deployments.

---

## Overview

All HTTP traffic flows through the gateway:

```
Client
  ↓
AuthGate Gateway
  ├── /auth/*  → AuthGate
  └── /*       → Application
```

---

## Quick Start (Docker)

```bash
docker run \
  -p 3000:3000 \
  -e GATEWAY_PORT=3000 \
  -e AUTHGATE_UPSTREAM=authgate:8080 \
  -e APP_UPSTREAM=app:3000 \
  ghcr.io/authgate/authgate-gateway:1.0.0
```

---

## Table of Contents

- Purpose
- Routing Model
- Environment Variables
- Example (Docker Compose)
- HTTPS / TLS
- Design Principles
- License

---

## Purpose

The gateway ensures that:

- applications never know where AuthGate lives
- authentication routes are fully centralized
- cookies and redirects stay on a single origin
- AuthGate remains **infrastructure**, not application logic

---

## Routing Model

All requests enter through the gateway:

```
/auth/*   → AuthGate
/*        → Application
```

Applications must redirect unauthenticated users to:

```
/auth/login
```

---

## Environment Variables

The gateway is **fully configured via environment variables**.

| Variable              | Required | Description                                  |
|-----------------------|----------|----------------------------------------------|
| `GATEWAY_PORT`        | No       | Port to listen on (default: `80`)             |
| `AUTHGATE_UPSTREAM`   | Yes      | AuthGate upstream (e.g. `authgate:8080`)      |
| `APP_UPSTREAM`        | Yes      | Application upstream (e.g. `app:3000`)        |

---

## Example (Docker Compose)

```yaml
services:
  gateway:
    image: ghcr.io/authgate/authgate-gateway:1.0.0
    ports:
      - "3000:3000"
    environment:
      GATEWAY_PORT: 3000
      AUTHGATE_UPSTREAM: authgate:8080
      APP_UPSTREAM: app:3000
    depends_on:
      - authgate
      - app

  authgate:
    image: ghcr.io/authgate/authgate-core:1.0.0
    ports:
      - "8080"

  app:
    image: example/app:latest
    ports:
      - "3000"
```

> This example shows **network wiring only**.  
> Application and AuthGate configuration are intentionally omitted.

---

## HTTPS / TLS

AuthGate Gateway does **not** terminate TLS by default.

HTTPS is expected to be handled by upstream infrastructure such as:

- Kubernetes Ingress
- Cloud load balancers
- Reverse proxies (NGINX, Traefik, Envoy)
- Corporate TLS gateways

This design keeps the gateway **portable and environment-agnostic**.

If TLS termination is required at the gateway, operators may provide a custom
Caddy configuration. TLS is intentionally disabled by default.

---

## License

MIT License
