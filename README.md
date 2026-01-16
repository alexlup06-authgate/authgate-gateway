# AuthGate Gateway

AuthGate Gateway is a small, config-driven reverse proxy used to route traffic
between an application and AuthGate.

It is based on Caddy and has **no environment-specific behavior**.

---

## Purpose

The gateway ensures:

- applications do not know where AuthGate lives
- authentication routes are centralized
- cookies and redirects stay on one origin
- AuthGate remains infrastructure, not application logic

---

## Routing Model

All traffic enters through the gateway.

```
/auth/*  → AuthGate
/*       → Application
```

Applications must redirect unauthenticated users to:

```
/auth/login
```

The gateway resolves the rest.

---

## Environment Variables

The gateway is fully configured via environment variables.

| Variable | Required | Description |
|--------|----------|-------------|
| `GATEWAY_PORT` | No | Port to listen on (default: 80) |
| `AUTHGATE_UPSTREAM` | Yes | AuthGate upstream (e.g. `authgate:8080`) |
| `APP_UPSTREAM` | Yes | Application upstream (e.g. `app:3000`) |

---

## Example (Docker)

```bash
docker run \
  -p 3000:3000 \
  -e GATEWAY_PORT=3000 \
  -e AUTHGATE_UPSTREAM=authgate:8080 \
  -e APP_UPSTREAM=app:3000 \
  ghcr.io/your-org/authgate-gateway:1.0.0
```

---

## HTTPS / TLS

AuthGate Gateway **does not terminate TLS by default**.

HTTPS is expected to be handled by upstream infrastructure such as:

- Kubernetes Ingress
- cloud load balancers
- reverse proxies (NGINX, Traefik, Envoy)
- corporate TLS gateways

This keeps the gateway portable and environment-agnostic.

If TLS termination is required at the gateway, operators may provide their own
Caddy configuration. TLS is intentionally not enabled by default.

---

## Design Principles

- no dev / prod modes
- no embedded routing logic
- no application awareness
- same image in all environments
- behavior driven entirely by configuration

AuthGate Gateway is infrastructure.

---

## License

MIT License
