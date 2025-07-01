# S-UI API Documentation

**An Advanced Web Panel • Built on SagerNet/Sing-Box**

This document provides comprehensive documentation for all public APIs, functions, and components in the S-UI project.

## Table of Contents

1. [Overview](#overview)
2. [Authentication](#authentication)
3. [REST API Endpoints](#rest-api-endpoints)
4. [Subscription API](#subscription-api)
5. [Core Components](#core-components)
6. [Service Layer](#service-layer)
7. [Data Models](#data-models)
8. [Utility Functions](#utility-functions)
9. [Configuration](#configuration)
10. [Examples](#examples)

## Overview

S-UI is a web-based management panel for SagerNet/Sing-Box with support for multiple protocols, advanced traffic routing, and comprehensive monitoring capabilities. The application provides both web UI and REST API access.

### Architecture

- **Frontend**: Vue.js based web interface
- **Backend**: Go/Gin HTTP server
- **Database**: SQLite with GORM ORM
- **Core**: SagerNet/Sing-Box integration
- **Protocols**: VLESS, VMess, Trojan, Shadowsocks, Hysteria, TUIC, and more

### Default Configuration

- **Panel Port**: 2095
- **Panel Path**: /app/
- **Subscription Port**: 2096
- **Subscription Path**: /sub/
- **Default Credentials**: admin/admin

## Authentication

### Session-based Authentication (Web UI)

The web UI uses session-based authentication with cookies.

#### Login
```
POST /app/api/login
Content-Type: application/x-www-form-urlencoded

user=username&pass=password
```

#### Logout
```
GET /app/api/logout
```

### Token-based Authentication (API v2)

API v2 endpoints require token authentication via the `Token` header.

#### Token Management
```
POST /app/api/addToken
Content-Type: application/x-www-form-urlencoded

expiry=1234567890&desc=API Token Description
```

```
POST /app/api/deleteToken
Content-Type: application/x-www-form-urlencoded

id=1
```

## REST API Endpoints

### API Version 1 (`/app/api/`)

All v1 endpoints require session authentication (login first).

#### System Operations

##### Get System Status
```
GET /app/api/status?r=cpu,mem,net,sys,sbd
```

**Parameters:**
- `r`: Comma-separated list of requested status types
  - `cpu`: CPU usage percentage
  - `mem`: Memory information
  - `net`: Network I/O statistics
  - `sys`: System information
  - `sbd`: Sing-box daemon status

**Response:**
```json
{
  "success": true,
  "msg": "status",
  "obj": {
    "cpu": 15.2,
    "mem": {
      "current": 1073741824,
      "total": 8589934592
    },
    "net": {
      "sent": 1234567890,
      "recv": 9876543210
    },
    "sys": {
      "cpuType": "Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz",
      "cpuCount": 12,
      "hostName": "s-ui-server",
      "appVersion": "1.7.0"
    },
    "sbd": {
      "running": true,
      "stats": {
        "NumGoroutine": 25,
        "Alloc": 12345678,
        "Uptime": 86400
      }
    }
  }
}
```

##### Get System Logs
```
GET /app/api/logs?c=100&l=info
```

**Parameters:**
- `c`: Number of log entries (default: 10)
- `l`: Log level filter (`debug`, `info`, `warn`, `error`)

##### Restart Application
```
POST /app/api/restartApp
```

##### Restart Sing-box Core
```
POST /app/api/restartSb
```

#### Configuration Management

##### Load All Configuration Data
```
GET /app/api/load?lu=1234567890
```

**Parameters:**
- `lu`: Last update timestamp (optional, for change detection)

**Response:**
```json
{
  "success": true,
  "msg": "load",
  "obj": {
    "config": {...},
    "clients": [...],
    "inbounds": [...],
    "outbounds": [...],
    "endpoints": [...],
    "tls": [...],
    "subURI": "https://domain.com:2096/sub/",
    "onlines": {...}
  }
}
```

##### Load Partial Configuration
```
GET /app/api/inbounds?id=1,2,3
GET /app/api/outbounds
GET /app/api/endpoints
GET /app/api/tls
GET /app/api/clients?id=1,2,3
GET /app/api/config
```

**Parameters:**
- `id`: Comma-separated list of IDs (for inbounds and clients)

##### Save Configuration
```
POST /app/api/save
Content-Type: application/x-www-form-urlencoded

object=clients&action=new&data={"name":"client1",...}&initUsers=1,2,3
```

**Parameters:**
- `object`: Configuration object type (`clients`, `inbounds`, `outbounds`, `endpoints`, `tls`, `config`, `settings`)
- `action`: Operation type (`new`, `edit`, `del`, `addbulk`)
- `data`: JSON data for the operation
- `initUsers`: Comma-separated client IDs (for inbound creation)

#### Client Management

##### Get All Clients
```
GET /app/api/clients
```

##### Get Specific Clients
```
GET /app/api/clients?id=1,2,3
```

##### Create New Client
```
POST /app/api/save
Content-Type: application/x-www-form-urlencoded

object=clients&action=new&data={"name":"client1","config":{"vless":{"uuid":"uuid-here"}},"inbounds":[1,2],"volume":10737418240,"expiry":1234567890,"desc":"Test client"}
```

##### Edit Client
```
POST /app/api/save
Content-Type: application/x-www-form-urlencoded

object=clients&action=edit&data={"id":1,"name":"updated-client",...}
```

##### Delete Client
```
POST /app/api/save
Content-Type: application/x-www-form-urlencoded

object=clients&action=del&data=1
```

##### Bulk Add Clients
```
POST /app/api/save
Content-Type: application/x-www-form-urlencoded

object=clients&action=addbulk&data=[{"name":"client1",...},{"name":"client2",...}]
```

#### Inbound Management

##### Get All Inbounds
```
GET /app/api/inbounds
```

##### Create New Inbound
```
POST /app/api/save
Content-Type: application/x-www-form-urlencoded

object=inbounds&action=new&data={"type":"vless","tag":"vless-in","listen":"0.0.0.0","listen_port":443,"tls":true}&initUsers=1,2,3
```

#### Statistics and Monitoring

##### Get Traffic Statistics
```
GET /app/api/stats?resource=client&tag=client1&limit=100
```

**Parameters:**
- `resource`: Resource type (`client`, `inbound`, `outbound`)
- `tag`: Resource tag/name
- `limit`: Maximum number of records

##### Get Online Connections
```
GET /app/api/onlines
```

##### Get Configuration Changes Log
```
GET /app/api/changes?a=admin&k=clients&c=50
```

**Parameters:**
- `a`: Actor (username)
- `k`: Change key (object type)
- `c`: Count limit

#### User Management

##### Get All Users
```
GET /app/api/users
```

##### Change Password
```
POST /app/api/changePass
Content-Type: application/x-www-form-urlencoded

id=1&oldPass=currentpass&newUsername=newuser&newPass=newpass
```

#### Settings Management

##### Get All Settings
```
GET /app/api/settings
```

#### Utilities

##### Generate Keypairs
```
GET /app/api/keypairs?k=reality&o=option1,option2
```

**Parameters:**
- `k`: Key type (`reality`, `tls`, `ech`, `wireguard`)
- `o`: Options specific to key type

##### Convert Proxy Links
```
POST /app/api/linkConvert
Content-Type: application/x-www-form-urlencoded

link=vless://uuid@host:port?security=tls#remark
```

##### Database Backup
```
GET /app/api/getdb?exclude=stats
```

**Parameters:**
- `exclude`: Tables to exclude from backup

##### Database Import
```
POST /app/api/importdb
Content-Type: multipart/form-data

db=[binary file data]
```

### API Version 2 (`/app/apiv2/`)

All v2 endpoints require token authentication via the `Token` header.

#### Available Endpoints

API v2 supports the same operations as v1 but with token-based authentication:

- `GET /app/apiv2/load`
- `GET /app/apiv2/inbounds`
- `GET /app/apiv2/outbounds`
- `GET /app/apiv2/endpoints`
- `GET /app/apiv2/tls`
- `GET /app/apiv2/clients`
- `GET /app/apiv2/config`
- `GET /app/apiv2/users`
- `GET /app/apiv2/settings`
- `GET /app/apiv2/stats`
- `GET /app/apiv2/status`
- `GET /app/apiv2/onlines`
- `GET /app/apiv2/logs`
- `GET /app/apiv2/changes`
- `GET /app/apiv2/keypairs`
- `GET /app/apiv2/getdb`
- `POST /app/apiv2/save`
- `POST /app/apiv2/restartApp`
- `POST /app/apiv2/restartSb`
- `POST /app/apiv2/linkConvert`
- `POST /app/apiv2/importdb`

#### Authentication Header
```
Token: your-api-token-here
```

## Subscription API

The subscription service provides client configuration links and JSON configurations.

### Base URL
```
http(s)://domain:2096/sub/
```

### Get Subscription Links
```
GET /sub/{client-id}
```

**Response:** Plain text list of proxy links (one per line)
```
vless://uuid@host:port?security=tls&sni=example.com#Server1
vmess://base64-encoded-config#Server2
trojan://password@host:port?security=tls#Server3
```

**Headers:**
- `Subscription-Userinfo`: Usage statistics
- `Profile-Update-Interval`: Update interval in hours
- `Profile-Title`: Profile name

### Get JSON Configuration
```
GET /sub/{client-id}?format=json
GET /sub/{client-id}?format=clash
GET /sub/{client-id}?format=singbox
```

**Parameters:**
- `format`: Output format (`json`, `clash`, `singbox`)

**Response:** JSON configuration compatible with the specified client

## Core Components

### Core Management

The core system manages the Sing-box instance.

#### Core Interface
```go
type Core struct {
    instance *Box
    config   *option.Options
    // ...
}

// Public methods
func (c *Core) Start(configData string) error
func (c *Core) Stop() error
func (c *Core) Restart(configData string) error
func (c *Core) IsRunning() bool
func (c *Core) GetInstance() *Box
func (c *Core) AddInbound(config []byte) error
func (c *Core) RemoveInbound(tag string) error
func (c *Core) AddOutbound(config []byte) error
func (c *Core) RemoveOutbound(tag string) error
```

### Box (Sing-box Instance)

The Box represents a running Sing-box instance.

#### Box Interface
```go
type Box struct {
    network     *route.NetworkManager
    endpoint    *endpoint.Manager
    inbound     *inbound.Manager
    outbound    *outbound.Manager
    connection  *route.ConnectionManager
    router      *route.Router
    // ...
}

// Public methods
func NewBox(options Options) (*Box, error)
func (s *Box) Start() error
func (s *Box) Close() error
func (s *Box) Uptime() uint32
func (s *Box) Network() adapter.NetworkManager
func (s *Box) Router() adapter.Router
func (s *Box) Inbound() adapter.InboundManager
func (s *Box) Outbound() adapter.OutboundManager
func (s *Box) Endpoint() adapter.EndpointManager
```

### Connection Tracking

#### ConnTracker Interface
```go
type ConnTracker struct {
    connections map[string]*Counter
    mutex       sync.RWMutex
}

type Counter struct {
    Up   int64
    Down int64
}

// Public methods
func NewConnTracker() *ConnTracker
func (ct *ConnTracker) AddCounter(tag string) *Counter
func (ct *ConnTracker) GetCounter(tag string) *Counter
func (ct *ConnTracker) GetCounters() map[string]*Counter
func (ct *ConnTracker) ResetCounter(tag string)
```

## Service Layer

### Client Service

Manages client configurations and operations.

```go
type ClientService struct {
    InboundService
}

// Public methods
func (s *ClientService) Get(id string) (*[]model.Client, error)
func (s *ClientService) GetAll() (*[]model.Client, error)
func (s *ClientService) Save(tx *gorm.DB, act string, data json.RawMessage, hostname string) ([]uint, error)
func (s *ClientService) DepleteClients() error
```

### Inbound Service

Manages inbound proxy configurations.

```go
type InboundService struct{}

// Public methods
func (s *InboundService) Get(ids string) (*[]map[string]interface{}, error)
func (s *InboundService) GetAll() (*[]map[string]interface{}, error)
func (s *InboundService) Save(tx *gorm.DB, act string, data json.RawMessage, initUserIds string, hostname string) (uint, error)
func (s *InboundService) GetAllConfig(db *gorm.DB) ([]json.RawMessage, error)
func (s *InboundService) RestartInbounds(tx *gorm.DB, ids []uint) error
```

### Outbound Service

Manages outbound proxy configurations.

```go
type OutboundService struct{}

// Public methods
func (s *OutboundService) GetAll() (*[]model.Outbound, error)
func (s *OutboundService) Save(tx *gorm.DB, act string, data json.RawMessage) error
func (s *OutboundService) GetAllConfig() ([]json.RawMessage, error)
```

### Setting Service

Manages system settings and configuration.

```go
type SettingService struct{}

// Public methods
func (s *SettingService) GetAllSetting() (*[]model.Setting, error)
func (s *SettingService) SaveSettings(settings []model.Setting) error
func (s *SettingService) GetConfig() (string, error)
func (s *SettingService) GetWebPath() (string, error)
func (s *SettingService) GetWebDomain() (string, error)
func (s *SettingService) GetSubPath() (string, error)
func (s *SettingService) GetSubDomain() (string, error)
func (s *SettingService) GetListen() (string, error)
func (s *SettingService) GetPort() (int, error)
func (s *SettingService) GetCertFile() (string, error)
func (s *SettingService) GetKeyFile() (string, error)
```

### Stats Service

Manages traffic statistics and monitoring.

```go
type StatsService struct{}

// Public methods
func (s *StatsService) SaveStats() error
func (s *StatsService) GetStats(resource string, tag string, limit int) ([]model.Stats, error)
func (s *StatsService) GetOnlines() (onlines, error)
func (s *StatsService) DelOldStats(days int) error
```

### Server Service

Provides system monitoring and server information.

```go
type ServerService struct{}

// Public methods
func (s *ServerService) GetStatus(request string) *map[string]interface{}
func (s *ServerService) GetCpuPercent() float64
func (s *ServerService) GetMemInfo() map[string]interface{}
func (s *ServerService) GetNetInfo() map[string]interface{}
func (s *ServerService) GetSystemInfo() map[string]interface{}
func (s *ServerService) GetLogs(count string, level string) []string
func (s *ServerService) GenKeypair(keyType string, options string) []string
```

### User Service

Manages user authentication and authorization.

```go
type UserService struct{}

// Public methods
func (s *UserService) GetFirstUser() (*model.User, error)
func (s *UserService) UpdateFirstUser(username string, password string) error
func (s *UserService) Login(username string, password string, remoteIP string) (string, error)
func (s *UserService) CheckUser(username string, password string, remoteIP string) *model.User
func (s *UserService) GetUsers() (*[]model.User, error)
func (s *UserService) ChangePass(id string, oldPass string, newUser string, newPass string) error
func (s *UserService) AddToken(username string, expiry int64, desc string) (string, error)
func (s *UserService) DeleteToken(id string) error
```

## Data Models

### Client Model

```go
type Client struct {
    Id       uint            `json:"id" gorm:"primaryKey;autoIncrement"`
    Enable   bool            `json:"enable"`
    Name     string          `json:"name"`
    Config   json.RawMessage `json:"config,omitempty"`
    Inbounds json.RawMessage `json:"inbounds"`
    Links    json.RawMessage `json:"links,omitempty"`
    Volume   int64           `json:"volume"`   // Traffic limit in bytes
    Expiry   int64           `json:"expiry"`   // Expiration timestamp
    Down     int64           `json:"down"`     // Downloaded bytes
    Up       int64           `json:"up"`       // Uploaded bytes
    Desc     string          `json:"desc"`     // Description
    Group    string          `json:"group"`    // Client group
}
```

**Client Configuration Examples:**

VLESS:
```json
{
  "vless": {
    "uuid": "uuid-string",
    "flow": "xtls-rprx-vision"
  }
}
```

VMess:
```json
{
  "vmess": {
    "uuid": "uuid-string",
    "security": "auto"
  }
}
```

Trojan:
```json
{
  "trojan": {
    "password": "password-string"
  }
}
```

Shadowsocks:
```json
{
  "shadowsocks": {
    "password": "password-string"
  }
}
```

### Inbound Model

```go
type Inbound struct {
    Id      uint            `json:"id" gorm:"primaryKey;autoIncrement"`
    Type    string          `json:"type"`    // Protocol type
    Tag     string          `json:"tag"`     // Unique identifier
    Options json.RawMessage `json:"options"` // Protocol-specific options
    OutJson json.RawMessage `json:"out_json,omitempty"`
    TlsId   uint            `json:"tls_id"`
    Tls     *Tls            `json:"tls,omitempty" gorm:"foreignKey:TlsId;references:Id"`
    Addrs   json.RawMessage `json:"addrs,omitempty"`
}
```

### Outbound Model

```go
type Outbound struct {
    Id      uint            `json:"id" gorm:"primaryKey;autoIncrement"`
    Type    string          `json:"type"`
    Tag     string          `json:"tag"`
    Options json.RawMessage `json:"options"`
}
```

### TLS Configuration Model

```go
type Tls struct {
    Id     uint            `json:"id" gorm:"primaryKey;autoIncrement"`
    Name   string          `json:"name"`
    Server json.RawMessage `json:"server"`
    Client json.RawMessage `json:"client"`
}
```

### Setting Model

```go
type Setting struct {
    Id    uint   `json:"id" gorm:"primaryKey;autoIncrement"`
    Key   string `json:"key"`
    Value string `json:"value"`
}
```

### User Model

```go
type User struct {
    Id         uint   `json:"id" gorm:"primaryKey;autoIncrement"`
    Username   string `json:"username"`
    Password   string `json:"password"`
    LastLogins string `json:"lastLogin"`
}
```

### Stats Model

```go
type Stats struct {
    Id        uint64 `json:"id" gorm:"primaryKey;autoIncrement"`
    DateTime  int64  `json:"dateTime"`
    Resource  string `json:"resource"`  // "client", "inbound", "outbound"
    Tag       string `json:"tag"`       // Resource identifier
    Direction bool   `json:"direction"` // true=up, false=down
    Traffic   int64  `json:"traffic"`   // Bytes
}
```

### Token Model

```go
type Tokens struct {
    Id     uint   `json:"id" gorm:"primaryKey;autoIncrement"`
    Desc   string `json:"desc"`
    Token  string `json:"token"`
    Expiry int64  `json:"expiry"`
    UserId uint   `json:"userId"`
    User   *User  `json:"user" gorm:"foreignKey:UserId;references:Id"`
}
```

## Utility Functions

### Link Generation

Generate proxy links from configurations.

```go
// Generate proxy links for a client
func LinkGenerator(clientConfig json.RawMessage, i *model.Inbound, hostname string) []string

// Supported protocols with link generation
var InboundTypeWithLink = []string{
    "shadowsocks", "naive", "hysteria", "hysteria2", 
    "tuic", "vless", "trojan", "vmess"
}
```

### Link Conversion

Convert proxy links to outbound configurations.

```go
// Convert proxy link to outbound configuration
func GetOutbound(uri string, i int) (*map[string]interface{}, string, error)
```

### Base64 Utilities

```go
// Encode string to base64 (with automatic detection)
func StrOrBase64Encoded(str string) string

// Decode base64 string to bytes
func B64StrToByte(str string) ([]byte, error)

// Encode bytes to base64 string
func ByteToB64Str(b []byte) string
```

### Random Generation

```go
// Generate random string
func Random(n int) string

// Generate random integer
func RandomInt(n int) int
```

### Error Handling

```go
// Create formatted error
func NewErrorf(format string, a ...interface{}) error

// Create simple error
func NewError(a ...interface{}) error

// Recover from panic
func Recover(msg string) interface{}
```

## Configuration

### Environment Variables

| Variable       | Type     | Default | Description                    |
|----------------|----------|---------|--------------------------------|
| SUI_LOG_LEVEL  | string   | "info"  | Log level (debug/info/warn/error) |
| SUI_DEBUG      | boolean  | false   | Enable debug mode              |
| SUI_BIN_FOLDER | string   | "bin"   | Binary files directory         |
| SUI_DB_FOLDER  | string   | "db"    | Database directory             |
| SINGBOX_API    | string   | -       | Sing-box API endpoint          |

### Default Settings

Settings are stored in the database and can be modified via the API:

- `webListen`: "0.0.0.0"
- `webPort`: "2095"
- `webPath`: "/app/"
- `webDomain`: ""
- `webCertFile`: ""
- `webKeyFile`: ""
- `subListen`: "0.0.0.0"
- `subPort`: "2096"
- `subPath`: "/sub/"
- `subDomain`: ""
- `subCertFile`: ""
- `subKeyFile`: ""
- `trafficAge`: "30"
- `timeLocation`: "Asia/Shanghai"
- `sessionMaxAge`: "60"

## Examples

### Complete Client Management Example

```bash
# 1. Login
curl -X POST http://localhost:2095/app/api/login \
  -d "user=admin&pass=admin" \
  -c cookies.txt

# 2. Create a new client
curl -X POST http://localhost:2095/app/api/save \
  -b cookies.txt \
  -d 'object=clients&action=new&data={"name":"test-client","config":{"vless":{"uuid":"550e8400-e29b-41d4-a716-446655440000","flow":"xtls-rprx-vision"}},"inbounds":[1],"volume":10737418240,"expiry":1735689600,"desc":"Test client for API example"}'

# 3. Get all clients
curl -X GET http://localhost:2095/app/api/clients \
  -b cookies.txt

# 4. Get client subscription
curl -X GET http://localhost:2096/sub/550e8400-e29b-41d4-a716-446655440000
```

### API v2 Token Example

```bash
# 1. Login and get token
curl -X POST http://localhost:2095/app/api/login \
  -d "user=admin&pass=admin" \
  -c cookies.txt

curl -X POST http://localhost:2095/app/api/addToken \
  -b cookies.txt \
  -d "expiry=0&desc=API Access Token"

# 2. Use token for API v2
curl -X GET http://localhost:2095/app/apiv2/clients \
  -H "Token: your-token-here"
```

### Subscription JSON Format Example

```bash
# Get Sing-box format configuration
curl "http://localhost:2096/sub/client-uuid?format=singbox"

# Response example:
{
  "outbounds": [
    {
      "type": "vless",
      "tag": "vless-server",
      "server": "example.com",
      "server_port": 443,
      "uuid": "550e8400-e29b-41d4-a716-446655440000",
      "flow": "xtls-rprx-vision",
      "tls": {
        "enabled": true,
        "server_name": "example.com",
        "alpn": ["h2", "http/1.1"]
      }
    }
  ]
}
```

### Link Conversion Example

```bash
# Convert proxy link to configuration
curl -X POST http://localhost:2095/app/api/linkConvert \
  -b cookies.txt \
  -d "link=vless://550e8400-e29b-41d4-a716-446655440000@example.com:443?security=tls&sni=example.com&alpn=h2,http/1.1&flow=xtls-rprx-vision#Server1"
```

### System Monitoring Example

```bash
# Get comprehensive system status
curl -X GET "http://localhost:2095/app/api/status?r=cpu,mem,net,sys,sbd" \
  -b cookies.txt

# Get traffic statistics
curl -X GET "http://localhost:2095/app/api/stats?resource=client&tag=test-client&limit=100" \
  -b cookies.txt

# Get online connections
curl -X GET http://localhost:2095/app/api/onlines \
  -b cookies.txt
```

### Bulk Client Creation Example

```bash
# Create multiple clients at once
curl -X POST http://localhost:2095/app/api/save \
  -b cookies.txt \
  -d 'object=clients&action=addbulk&data=[
    {
      "name":"client1",
      "config":{"vless":{"uuid":"uuid1","flow":"xtls-rprx-vision"}},
      "inbounds":[1],
      "volume":5368709120,
      "expiry":1735689600,
      "desc":"Bulk client 1"
    },
    {
      "name":"client2", 
      "config":{"vless":{"uuid":"uuid2","flow":"xtls-rprx-vision"}},
      "inbounds":[1],
      "volume":5368709120,
      "expiry":1735689600,
      "desc":"Bulk client 2"
    }
  ]'
```

This documentation covers all public APIs, functions, and components in the S-UI project. For additional information, refer to the source code or the project's GitHub repository.