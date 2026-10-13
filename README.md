# Gaty 🚪

A tiny API gateway for my game backend.

One public entry point. A couple of microservices behind it. That's pretty much the idea.

Built around YARP with HTTP/2 forwarding and JWT authorization.

## 🛠️ Tech Stack

[![.NET](https://img.shields.io/badge/10-512BD4?style=for-the-badge&logo=dotnet&logoColor=white)](https://github.com/dotnet/runtime)
[![ASP.NET Core](https://img.shields.io/badge/ASP.NET_Core-512BD4?style=for-the-badge&logo=dotnet&logoColor=white)](https://github.com/dotnet/aspnetcore)
[![YARP](https://img.shields.io/badge/YARP-Reverse_Proxy-512BD4?style=for-the-badge&logo=dotnet&logoColor=white)](https://github.com/dotnet/yarp)

## ⚡ What's Inside

- **Reverse Proxy** — YARP forwards requests to backend microservices.
- **HTTP/2** — downstream requests keep the HTTP/2 transport required by the gRPC services.
- **JWT Validation** — protected routes are authenticated at the gateway.
- **Route Policies** — each microservice can be public or protected independently.
- **Path Transforms** — public service prefixes are removed before forwarding downstream.
- **Configuration-Based Routing** — routes and service destinations live in configuration rather than application code.

## 🧩 Services

```text
                            ┌─────────────┐
                            │   Client    │
                            └──────┬──────┘
                                   │
                                   ▼
                            ┌─────────────┐
                            │    Gaty     │
                            │             │
                            │ JWT Auth    │
                            │ Routing     │
                            │ HTTP/2      │
                            └───┬─────┬───┘
                                │     │
                    anonymous   │     │   authorized
                                │     │
                         /authy │     │ /aucy
                                ▼     ▼
                         ┌────────┐ ┌────────┐
                         │ Authy  │ │  Aucy  │
                         └────────┘ └────────┘
```

### `/authy/*`

Forwarded to [**Authy**](https://github.com/Quaximur/Authy).

Authentication endpoints are available anonymously so clients can register and obtain a JWT.

```text
/authy/...  ──►  Authy /...
```

### `/aucy/*`

Forwarded to [**Aucy**](https://github.com/Quaximur/Aucy).

The route requires a valid JWT before the request reaches the auction service.

```text
/aucy/...  ── JWT ──►  Aucy /...
```

## 🔐 Authentication Flow

```text
Client
  │
  │  Register / Login
  ▼
Gaty ─────────────────► Authy
  ▲                       │
  │         JWT           │
  └───────────────────────┘
  │
  │  Request + JWT
  ▼
Gaty
  │
  │  Validate JWT
  ▼
Aucy
```

Authy issues the token. Gaty validates it. Aucy can then trust the authenticated `user_id` forwarded through the request context.

## 🚀 Running

Configure the same JWT secret used by [Authy](https://github.com/Quaximur/Authy), then run:

```bash
dotnet run --project API/Gaty.API.csproj
```

By default, Gaty exposes the backend through a single gateway while forwarding requests to the individual services over HTTP/2.
