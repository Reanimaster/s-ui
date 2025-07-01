# S-UI – Comprehensive Backend Documentation

## Table of Contents
1. Overview and Architecture
2. Build / Run / Deployment
3. HTTP API Reference
4. Subscription Service (`/sub`) Reference
5. Package & Public Function Reference (Go)
6. Examples
7. Appendix – Glossary & Conventions

---

## 1. Overview and Architecture
S-UI is an advanced web panel built around the [sing-box](https://sing-box.sagernet.org) core.  
The **backend** is implemented in Go and is split into logical packages:

* `api`  – REST-like HTTP handlers exposed under `/api` and `/apiv2`
* `service`  – Business-logic layer (settings, users, TLS, in/out-bounds …)
* `database`  – SQLite access layer and data models
* `core`  – Thin wrapper that starts/stops **sing-box** and tracks its state
* `web` + `sub`  – Two HTTP servers (Web UI & Subscription service) built with `gin`
* `cronjob`  – Scheduled background jobs (traffic purge, stats aggregation …)
* `util`, `middleware`, `network`, `logger`  – Helpers shared by multiple layers

High-level data–flow
```
┌──────────┐      ┌──────────┐      ┌──────────┐
│  Client  │ ───▶ │  /api    │ ───▶ │ service  │ ─┐
└──────────┘      └──────────┘      └──────────┘  │ writes
                                 ▲                ▼
                                 │           ┌──────────┐
                                 │           │ database │
                                 │           └──────────┘
                                 │                ▲
                                 │                │ generates
                                 └─────────┬──────┘
                                           ▼
                                      ┌────────┐
                                      │ core   │ → sing-box process
                                      └────────┘
```

The **frontend** (not included in this repository) is pre-built and served as static files by the `web` server.

---

## 2. Build / Run / Deployment
See `README.md` for full instructions.  Below is the minimal local workflow:

```bash
# 1. Clone & build
$ git clone https://github.com/alireza0/s-ui.git && cd s-ui
$ go build -o sui main.go

# 2. Start backend (uses ports defined in DB‐backed settings)
$ ./sui
```

Docker image is published as `alireza7/s-ui` and can be run with:
```bash
$ docker run -itd -p 2095:2095 -p 2096:2096 alireza7/s-ui:latest
```

---

## 3. HTTP API Reference
All paths below are **relative to the configured `BASE_URL`** (default `/app/`) followed by the prefix `api/` for v1 or `apiv2/` for token-based v2.

Authentication:
* **v1** – cookie-based session. Obtain with `POST /api/login`.
* **v2** – token passed through request header `Token: <tokenValue>` (obtain via `/api/addToken`).

### 3.1 `GET` End-points
| Path | Query / URL Params | Description |
|------|-------------------|-------------|
| `load` | `lu=<unix>` | Load full dashboard data; `lu` (=lastUpdate) allows delta refresh. |
| `inbounds` | `id=<inboundId>` | Get single inbound (v2 identical). |
| `outbounds` | – | List all outbounds |
| `endpoints` | – | List all endpoints |
| `tls` | – | List TLS configs |
| `clients` | `id=<clientId>` | Get client(s). |
| `config` | – | Download full sing-box JSON config |
| `users` | – | List admin users |
| `settings` | – | Current panel settings |
| `stats` | `resource, tag, limit` | Traffic / connection stats |
| `status` | `r=<statusKey>` | sing-box health & version |
| `onlines` | – | Currently online clients |
| `logs` | `c=<count>, l=<level>` | Tail sing-box logs |
| `changes` | `a=<actor>, k=<key>, c=<count>` | Config-change history |
| `keypairs` | `k=<type>, o=<options>` | Generate sing-box key-pair |
| `getdb` | `exclude=<commaSeparatedTables>` | Download SQLite database backup |
| `tokens` (v1 only) | – | List existing API tokens |

### 3.2 `POST` End-points  
(Same relative path as above)
| Path | Form fields | Purpose |
|------|-------------|---------|
| `login` | `user`, `pass` | Create session cookie |
| `logout` (`GET`) | – | Destroy session |
| `changePass` | `id`, `oldPass`, `newUsername`, `newPass` | Change user credentials |
| `save` | `object`, `action`, `data`, `initUsers` | Persist changes (`object ∈ {clients,tls,inbounds,…}`) |
| `restartApp` | – | Soft-restart S-UI panel |
| `restartSb` | – | Restart sing-box core |
| `linkConvert` | `link` | Convert raw share link → outbound JSON |
| `importdb` | `db=<SQLite file>` | Replace database (⚠ dangerous) |
| `addToken` | `expiry=<unix>`, `desc` | Create API token (v1 only) |
| `deleteToken` | `id` | Delete API token (v1 only) |

> **Sample – fetch dashboard data**
> ```bash
> curl -b cookie.txt \
>      http://localhost:2095/app/api/load?lu=0
> ```


---

## 4. Subscription Service (`/sub`)
The second HTTP server exposes subscription data for clients (ShadowSocks, Clash, sing-box …).

```
GET /sub/{subscription-id}[?format=json]
```
* When `format=json` is absent – returns a **base64-encoded** multi-protocol text list.
* When present – returns JSON suitable for Clash / sing-box GUI.

Response adds the following headers:
* `Subscription-Userinfo: <upload=0;download=0;total=…>`
* `Profile-Update-Interval: <seconds>`
* `Profile-Title: <remark>`

Example:
```bash
curl http://localhost:2096/sub/abcd1234?format=json
```

---

## 5. Package & Public Function Reference
Below is the distilled list of **exported** symbols you will commonly use when hacking on the backend.  See the code or run `go doc <pkg>` for full signatures.

### 5.1 `service` Package
* `NewConfigService(core *core.Core) *ConfigService` – facade that coordinates multiple sub-services and controls sing-box.
* `ConfigService.StartCore / StopCore / RestartCore`
* `ConfigService.Save` – transactional save helper; decides which sub-service should handle the object.
* `ClientService` – CRUD helpers & share-link generation.
* `InboundService / OutboundService / TlsService / EndpointService` – CRUD + config builders.
* `StatsService.GetStats / GetOnlines` – traffic metrics.
* `SettingService` – strongly-typed accessors for all panel settings (port, domain, paths, …).

### 5.2 `api` Package
* `NewAPIHandler(g *gin.RouterGroup, a2 *APIv2Handler)` – cookie-based v1 router.
* `NewAPIv2Handler(g *gin.RouterGroup)` – token-based router.
* `ApiService` – shared logic for both versions (`LoadData`, `Save`, `Login`, …).

### 5.3 `core` Package
* `NewCore() *Core` – starts isolated sing-box process.
* `Core.Start / Stop / Restart`  – control lifecycle.

### 5.4 `web` & `sub`
* `web.NewServer() *Server` – serves UI & API.
* `sub.NewServer() *Server` – serves subscription endpoint.

### 5.5 Utilities of Interest
* `util.LinkGenerator` – build share links for multiple protocols.
* `util.GetOutbound` – parse a share link into outbound JSON.
* `util.common.Random / RandomInt` – cryptographically secure random helpers.
* `network.NewAutoHttpsListener` – transparent HTTPS upgrade (port-443 autoprobe).

---

## 6. Examples
### 6.1 Programmatic Config Reload
```go
import (
    "s-ui/core"
    "s-ui/service"
)

c := core.NewCore()
cs := service.NewConfigService(c)
if err := cs.RestartCore(); err != nil {
    log.Fatal(err)
}
```

### 6.2 Generate a share link for a new client
```go
links := util.LinkGenerator(client.OutJson, inboundPtr, "example.com")
fmt.Println("Share URI:", links[0])
```

### 6.3 Using the v2 API from Bash
```bash
# 1. Authenticate (create token)
curl -b cookie.txt -F "expiry=$(date -d "+30 days" +%s)" -F "desc=mobile" \
     http://localhost:2095/app/api/addToken

# 2. Copy the returned token and use in header
token=XXXXXXXXXXXX
curl -H "Token: $token" http://localhost:2095/app/apiv2/load
```

---

## 7. Appendix – Glossary & Conventions
* **Inbound** – Listens for connections from clients.
* **Outbound** – Dial-out transport used by sing-box.
* **Endpoint** – Additional hosts reachable from rules.
* **TLS Config** – Reusable certificate + handshake options.
* **Client** – End-user credential (uuid/password/public-key …).
* **Tag** – Unique string identifier which ties pieces together.

When the documentation mentions **`BASE_URL`** it refers to the Web-UI path stored in settings (default `/app/`).

---

> Generated automatically – last update: 2025-07-01