# S-UI API Documentation

## Table of Contents
1. [Overview](#overview)
2. [Authentication](#authentication)
3. [API Endpoints](#api-endpoints)
   - [v1 API Endpoints](#v1-api-endpoints)
   - [v2 API Endpoints](#v2-api-endpoints)
4. [Core Components](#core-components)
5. [Services](#services)
6. [Utility Functions](#utility-functions)
7. [Models and Data Structures](#models-and-data-structures)
8. [Examples](#examples)

## Overview

S-UI is an advanced web panel built on SagerNet/Sing-Box that provides a comprehensive API for managing proxy configurations, traffic routing, and system monitoring. The project is built with Go and uses Gin web framework for REST API endpoints.

### Base URLs
- **Web Panel**: `http://localhost:2095/app/`
- **API v1**: `http://localhost:2095/app/api/`
- **API v2**: `http://localhost:2095/app/apiv2/`
- **Subscription Service**: `http://localhost:2096/sub/`

## Authentication

### Session-based Authentication (API v1)
The v1 API uses session-based authentication with cookies.

#### Login
```
POST /api/login
Content-Type: application/x-www-form-urlencoded

user=admin&pass=admin
```

#### Logout
```
GET /api/logout
```

### Token-based Authentication (API v2)
The v2 API uses header-based token authentication.

```
GET /apiv2/load
Headers:
  Token: <your-api-token>
```

## API Endpoints

### v1 API Endpoints

#### POST Endpoints
| Endpoint | Description | Parameters |
|----------|-------------|------------|
| `/api/login` | User login | `user`, `pass` |
| `/api/changePass` | Change user password | `id`, `oldPass`, `newUsername`, `newPass` |
| `/api/save` | Save configuration | `object`, `action`, `data` |
| `/api/restartApp` | Restart application | None |
| `/api/restartSb` | Restart Sing-Box core | None |
| `/api/linkConvert` | Convert subscription link | `link` |
| `/api/importdb` | Import database | Form file: `db` |
| `/api/addToken` | Add API token | `expiry`, `desc` |
| `/api/deleteToken` | Delete API token | `id` |

#### GET Endpoints
| Endpoint | Description | Query Parameters |
|----------|-------------|------------------|
| `/api/logout` | User logout | None |
| `/api/load` | Load all data | `lu` (last update) |
| `/api/inbounds` | Get inbounds | `id` (optional) |
| `/api/outbounds` | Get outbounds | None |
| `/api/endpoints` | Get endpoints | None |
| `/api/tls` | Get TLS configs | None |
| `/api/clients` | Get clients | `id` (optional) |
| `/api/config` | Get configuration | None |
| `/api/users` | Get all users | None |
| `/api/settings` | Get all settings | None |
| `/api/stats` | Get statistics | `resource`, `tag`, `limit` |
| `/api/status` | Get system status | `r` (request type) |
| `/api/onlines` | Get online clients | None |
| `/api/logs` | Get logs | `c` (count), `l` (level) |
| `/api/changes` | Check changes | `a` (actor), `k` (key), `c` (count) |
| `/api/keypairs` | Generate keypairs | `k` (type), `o` (options) |
| `/api/getdb` | Download database | `exclude` |
| `/api/tokens` | Get user tokens | None |

### v2 API Endpoints

The v2 API provides the same functionality as v1 but with token-based authentication. All endpoints require a `Token` header.

#### POST Endpoints
- `/apiv2/save`
- `/apiv2/restartApp`
- `/apiv2/restartSb`
- `/apiv2/linkConvert`
- `/apiv2/importdb`

#### GET Endpoints
- `/apiv2/load`
- `/apiv2/inbounds`
- `/apiv2/outbounds`
- `/apiv2/endpoints`
- `/apiv2/tls`
- `/apiv2/clients`
- `/apiv2/config`
- `/apiv2/users`
- `/apiv2/settings`
- `/apiv2/stats`
- `/apiv2/status`
- `/apiv2/onlines`
- `/apiv2/logs`
- `/apiv2/changes`
- `/apiv2/keypairs`
- `/apiv2/getdb`

## Core Components

### Application (app.App)
The main application controller that manages the lifecycle of all components.

```go
// Create new application instance
app := app.NewApp()

// Initialize application
err := app.Init()

// Start application
err := app.Start()

// Stop application
app.Stop()

// Restart application
app.RestartApp()
```

### Web Server (web.Server)
Handles the web interface and API endpoints.

```go
// Create new web server
server := web.NewServer()

// Start server
err := server.Start()

// Stop server
err := server.Stop()
```

### Subscription Server (sub.Server)
Manages subscription services for client configurations.

```go
// Create new subscription server
server := sub.NewServer()

// Start server
err := server.Start()

// Stop server
err := server.Stop()
```

### Core Engine (core.Core)
The Sing-Box core engine wrapper.

```go
// Create new core instance
core := core.NewCore()

// Create box with options
box, err := core.NewBox(options)

// Start core
err := box.Start()

// Stop core
err := box.Close()
```

## Services

### ConfigService
Manages configuration and core operations.

```go
type ConfigService struct {
    // Methods
    GetConfig(data string) (*SingBoxConfig, error)
    Save(obj string, act string, data json.RawMessage, initUsers string, loginUser string, hostname string) ([]string, error)
    RestartCore() error
    CheckChanges(lastUpdate string) (bool, error)
    GetChanges(actor string, chngKey string, count string) []model.Changes
}
```

### ClientService
Manages client configurations.

```go
type ClientService struct {
    // Methods
    Get(id string) (*[]model.Client, error)
    GetAll() (*[]model.Client, error)
    Save(data *[]model.Client) error
    Del(ids []int) error
}
```

### InboundService
Manages inbound connections.

```go
type InboundService struct {
    // Methods
    Get(ids string) (*[]map[string]interface{}, error)
    GetAll() (*[]map[string]interface{}, error)
    Save(data []byte) error
    Del(ids []string) error
}
```

### OutboundService
Manages outbound connections.

```go
type OutboundService struct {
    // Methods
    GetAll() (*[]map[string]interface{}, error)
    Save(data []byte) error
    Del(ids []string) error
}
```

### UserService
Manages user authentication and tokens.

```go
type UserService struct {
    // Methods
    GetFirstUser() (*model.User, error)
    GetUsers() (*[]model.User, error)
    Login(username string, password string, remoteIP string) (string, error)
    ChangePass(id string, oldPass string, newUsername string, newPass string) error
    GetUserTokens(username string) (*[]model.Tokens, error)
    AddToken(username string, expiry int64, desc string) (*model.Tokens, error)
    DeleteToken(tokenId string) error
}
```

### StatsService
Manages traffic statistics.

```go
type StatsService struct {
    // Methods
    GetStats(resource string, tag string, limit int) ([]model.Stats, error)
    GetOnlines() (onlines, error)
    GetClientStats(username string) (*model.ClientStats, error)
}
```

### SettingService
Manages application settings.

```go
type SettingService struct {
    // Methods
    GetAllSetting() (*map[string]string, error)
    GetListen() (string, error)
    GetPort() (int, error)
    GetWebPath() (string, error)
    GetCertFile() (string, error)
    GetKeyFile() (string, error)
    GetConfig() (string, error)
    // ... many more setting getters
}
```

## Utility Functions

### Link Generation (util/genLink.go)
Generate client configuration links for various protocols.

```go
// Generate VLESS link
link := genLink.GenVless(username, uuid, address, port, params)

// Generate VMess link
link := genLink.GenVmess(username, uuid, address, port, params)

// Generate Trojan link
link := genLink.GenTrojan(username, password, address, port, params)

// Generate Shadowsocks link
link := genLink.GenShadowsocks(username, method, password, address, port, params)
```

### Link to JSON Conversion (util/linkToJson.go)
Convert subscription links to JSON configurations.

```go
// Get outbound configuration from link
outbound, name, err := util.GetOutbound(link, index)
```

### Base64 Operations (util/base64.go)
```go
// Encode to base64
encoded := util.Base64Encode(data)

// Decode from base64
decoded, err := util.Base64Decode(encoded)
```

## Models and Data Structures

### User Model
```go
type User struct {
    Id       int    `json:"id" gorm:"primaryKey;autoIncrement"`
    Username string `json:"username"`
    Password string `json:"password"`
}
```

### Client Model
```go
type Client struct {
    Id            int    `json:"id" gorm:"primaryKey;autoIncrement"`
    Enable        bool   `json:"enable"`
    Name          string `json:"name" gorm:"unique"`
    Password      string `json:"password"`
    Flow          string `json:"flow"`
    LimitIP       int    `json:"limitIp"`
    TotalGB       int64  `json:"totalGB"`
    ExpiryTime    int64  `json:"expiryTime"`
    TrafficUsage  string `json:"trafficUsage"`
}
```

### Stats Model
```go
type Stats struct {
    Id       int    `json:"id" gorm:"primaryKey;autoIncrement"`
    Resource string `json:"resource"`
    Tag      string `json:"tag"`
    Traffic  int64  `json:"traffic"`
    Date     int64  `json:"date"`
}
```

### Token Model
```go
type Tokens struct {
    Id       int    `json:"id" gorm:"primaryKey;autoIncrement"`
    Token    string `json:"token"`
    Username string `json:"username"`
    Expiry   int64  `json:"expiry"`
    Desc     string `json:"desc"`
}
```

## Examples

### 1. Authentication and Basic Usage

#### Login (v1 API)
```bash
# Login
curl -X POST http://localhost:2095/app/api/login \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "user=admin&pass=admin" \
  -c cookies.txt

# Load data with session
curl http://localhost:2095/app/api/load \
  -b cookies.txt
```

#### Using Token Authentication (v2 API)
```bash
# First, add a token via v1 API
curl -X POST http://localhost:2095/app/api/addToken \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "expiry=1735689600&desc=API%20Access" \
  -b cookies.txt

# Use the token with v2 API
curl http://localhost:2095/app/apiv2/load \
  -H "Token: your-token-here"
```

### 2. Managing Clients

#### Get All Clients
```bash
curl http://localhost:2095/app/api/clients \
  -b cookies.txt
```

#### Add New Client
```bash
curl -X POST http://localhost:2095/app/api/save \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'object=client&action=add&data=[{"name":"test_user","enable":true,"password":"test123","limitIp":3,"totalGB":10737418240,"expiryTime":1735689600}]' \
  -b cookies.txt
```

### 3. Managing Inbounds

#### Get All Inbounds
```bash
curl http://localhost:2095/app/api/inbounds \
  -b cookies.txt
```

#### Add VLESS Inbound
```bash
curl -X POST http://localhost:2095/app/api/save \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d 'object=inbound&action=add&data=[{"type":"vless","tag":"vless-in","listen":"0.0.0.0","listen_port":8443,"tcp_fast_open":false,"udp_fragment":false,"sniff":true,"sniff_override_destination":true,"domain_strategy":"prefer_ipv4","tls":{"enabled":true,"server_name":"example.com","alpn":["h2","http/1.1"],"certificate_path":"/root/cert/cert.pem","key_path":"/root/cert/key.pem"}}]' \
  -b cookies.txt
```

### 4. System Operations

#### Get System Status
```bash
curl http://localhost:2095/app/api/status?r=all \
  -b cookies.txt
```

Response:
```json
{
  "cpu": 15.5,
  "mem": {
    "current": 1234567890,
    "total": 8589934592
  },
  "disk": {
    "current": 12345678900,
    "total": 107374182400
  },
  "uptime": 3600,
  "loads": [1.5, 1.2, 0.9]
}
```

#### Get Traffic Statistics
```bash
curl "http://localhost:2095/app/api/stats?resource=client&tag=test_user&limit=100" \
  -b cookies.txt
```

#### Get Online Clients
```bash
curl http://localhost:2095/app/api/onlines \
  -b cookies.txt
```

### 5. Subscription Service

#### Get Client Subscription
```bash
# Link format
curl http://localhost:2096/sub/client_name/link

# JSON format
curl http://localhost:2096/sub/client_name/json
```

### 6. Core Operations

#### Restart Sing-Box Core
```bash
curl -X POST http://localhost:2095/app/api/restartSb \
  -b cookies.txt
```

#### Restart Application
```bash
curl -X POST http://localhost:2095/app/api/restartApp \
  -b cookies.txt
```

### 7. Database Operations

#### Export Database
```bash
curl http://localhost:2095/app/api/getdb \
  -b cookies.txt \
  -o backup.db
```

#### Import Database
```bash
curl -X POST http://localhost:2095/app/api/importdb \
  -F "db=@backup.db" \
  -b cookies.txt
```

### 8. Advanced Configuration

#### Generate Keypairs
```bash
# Generate Reality keypair
curl "http://localhost:2095/app/api/keypairs?k=reality" \
  -b cookies.txt

# Generate WireGuard keypair
curl "http://localhost:2095/app/api/keypairs?k=wg" \
  -b cookies.txt
```

#### Convert Subscription Link
```bash
curl -X POST http://localhost:2095/app/api/linkConvert \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "link=vless://uuid@host:port?params" \
  -b cookies.txt
```

## Error Handling

All API endpoints return JSON responses with the following structure:

### Success Response
```json
{
  "success": true,
  "msg": "",
  "obj": {
    // Response data
  }
}
```

### Error Response
```json
{
  "success": false,
  "msg": "Error message",
  "obj": null
}
```

## WebSocket Support

The application supports WebSocket connections for real-time updates:
- System status monitoring
- Traffic statistics
- Online client monitoring

## Rate Limiting

Currently, there are no built-in rate limits. It's recommended to implement rate limiting at the reverse proxy level if exposing the API publicly.

## Security Considerations

1. **Change Default Credentials**: Always change the default admin/admin credentials
2. **Use HTTPS**: Configure SSL certificates for production deployments
3. **Token Security**: Keep API tokens secure and rotate them regularly
4. **Network Security**: Restrict access to the API endpoints using firewall rules
5. **Database Backups**: Regularly backup your database using the export functionality

## Development

### Building from Source
```bash
# Clone repository
git clone https://github.com/alireza0/s-ui
cd s-ui

# Clone submodules
git submodule update --init --recursive

# Build frontend
cd frontend
npm install
npm run build
cd ..

# Copy frontend files
rm -fr web/html/*
cp -R frontend/dist/ web/html/

# Build backend
go build -o sui main.go
```

### Running in Development Mode
```bash
# Set environment variables
export SUI_DEBUG=true
export SUI_LOG_LEVEL=debug

# Run application
./sui
```

## Contributing

When contributing to the API:
1. Follow the existing code structure
2. Add appropriate error handling
3. Update this documentation for new endpoints
4. Write tests for new functionality
5. Ensure backward compatibility

## Support

For issues and questions:
- GitHub Issues: https://github.com/alireza0/s-ui/issues
- Documentation: https://github.com/alireza0/s-ui/wiki