# S-UI Component Documentation

## Core Components

### 1. Logger Component

The logger component provides structured logging throughout the application.

#### Usage
```go
import "s-ui/logger"

// Initialize logger with log level
logger.InitLogger(logging.DEBUG)

// Log messages
logger.Debug("Debug message")
logger.Info("Info message")
logger.Warning("Warning message")
logger.Error("Error message")
logger.Errorf("Error with format: %v", err)
```

### 2. Database Component

The database component handles all data persistence using GORM.

#### Initialization
```go
import "s-ui/database"

// Initialize database
err := database.InitDB("/path/to/s-ui.db")

// Get database connection
db := database.GetDB()
```

#### Backup and Restore
```go
// Export database
data, err := database.GetDb("exclude_pattern")

// Import database
err := database.ImportDB(reader)
```

### 3. Middleware Components

#### Domain Validator
Validates requests against configured domain restrictions.

```go
import "s-ui/middleware"

// Use in Gin router
router.Use(middleware.DomainValidator("allowed-domain.com"))
```

### 4. Network Components

#### Auto HTTPS
Automatically detects and handles HTTPS connections.

```go
import "s-ui/network"

// Wrap listener for auto HTTPS
listener = network.NewAutoHttpsListener(listener)

// Wrap connection for auto HTTPS
conn = network.NewAutoHttpsConn(conn)
```

### 5. Cronjob Component

Manages scheduled tasks for the application.

```go
import "s-ui/cronjob"

// Create new cronjob manager
cron := cronjob.NewCronJob()

// Start cronjobs
err := cron.Start(location, trafficAge)

// Stop cronjobs
cron.Stop()
```

Available jobs:
- **StatsJob**: Collects traffic statistics
- **DepleteJob**: Monitors client usage limits
- **DelStatsJob**: Cleans old statistics
- **CheckCoreJob**: Monitors Sing-Box core status

## Service Components

### 1. Setting Service

Manages all application settings stored in the database.

#### Common Settings
```go
// Web panel settings
listen, err := settingService.GetListen()          // Default: ""
port, err := settingService.GetPort()             // Default: 2095
webPath, err := settingService.GetWebPath()       // Default: "/app/"
certFile, err := settingService.GetCertFile()     // SSL certificate
keyFile, err := settingService.GetKeyFile()       // SSL key

// Subscription settings
subListen, err := settingService.GetSubListen()   // Default: ""
subPort, err := settingService.GetSubPort()       // Default: 2096
subPath, err := settingService.GetSubPath()       // Default: "/sub/"

// Security settings
secret, err := settingService.GetSecret()         // Session secret
sessionMaxAge, err := settingService.GetSessionMaxAge()
```

### 2. Config Service

Manages Sing-Box configuration and core operations.

#### Configuration Management
```go
// Get current configuration
config, err := configService.GetConfig("")

// Save configuration changes
objs, err := configService.Save(object, action, data, initUsers, loginUser, hostname)

// Restart core
err := configService.RestartCore()

// Check for configuration changes
isUpdated, err := configService.CheckChanges(lastUpdate)
```

### 3. Stats Service

Manages traffic statistics and online client monitoring.

#### Statistics Operations
```go
// Get traffic statistics
stats, err := statsService.GetStats("client", "username", 100)

// Get online clients
onlines, err := statsService.GetOnlines()

// Get client statistics
clientStats, err := statsService.GetClientStats("username")
```

### 4. Server Service

Provides system information and core status.

#### System Information
```go
// Get system status
status := serverService.GetStatus("all")

// Get CPU usage
cpu := serverService.GetCpuPercent()

// Get memory information
memInfo := serverService.GetMemInfo()

// Get network information
netInfo := serverService.GetNetInfo()

// Get Sing-Box status
sbInfo := serverService.GetSingboxInfo()

// Get logs
logs := serverService.GetLogs("100", "info")
```

#### Keypair Generation
```go
// Generate Reality keypair
keypair := serverService.GenKeypair("reality", "")

// Generate WireGuard keypair
keypair := serverService.GenKeypair("wg", "")

// Generate X25519 keypair
keypair := serverService.GenKeypair("x25519", "")
```

### 5. Panel Service

Controls the application lifecycle.

```go
// Restart panel with delay (seconds)
err := panelService.RestartPanel(3)

// Get panel status
isRunning := panelService.IsRunning()
```

## Utility Functions

### 1. Link Generation

Generate client configuration links for various protocols.

#### VLESS
```go
params := map[string]string{
    "security": "tls",
    "type": "tcp",
    "sni": "example.com",
}
link := genLink.GenVless("client_name", "uuid", "server.com", 443, params)
```

#### VMess
```go
params := map[string]string{
    "aid": "0",
    "net": "ws",
    "path": "/path",
}
link := genLink.GenVmess("client_name", "uuid", "server.com", 443, params)
```

#### Trojan
```go
params := map[string]string{
    "security": "tls",
    "type": "tcp",
    "sni": "example.com",
}
link := genLink.GenTrojan("client_name", "password", "server.com", 443, params)
```

#### Shadowsocks
```go
params := map[string]string{
    "type": "tcp",
}
link := genLink.GenShadowsocks("client_name", "aes-256-gcm", "password", "server.com", 8388, params)
```

### 2. Link Parsing

Parse configuration links to JSON format.

```go
// Parse any supported link type
outbound, name, err := util.GetOutbound("vless://...", 0)

// The outbound will be a map[string]interface{} containing:
// - type: Protocol type (vless, vmess, trojan, shadowsocks, etc.)
// - tag: Outbound tag
// - server: Server address
// - server_port: Server port
// - Additional protocol-specific fields
```

### 3. Common Utilities

#### Error Handling
```go
import "s-ui/util/common"

// Create formatted error
err := common.NewErrorf("Operation failed: %s", reason)

// Create simple error
err := common.NewError("Operation failed")
```

## Subscription Service

The subscription service provides client configurations in multiple formats.

### Endpoints
- `GET /sub/{client}/link` - Get subscription links (base64 encoded)
- `GET /sub/{client}/json` - Get JSON configuration
- `GET /sub/{client}/qr` - Get QR codes (if enabled)

### Link Service
```go
// Generate client links
links, err := linkService.GetLinks(clientName)

// Get all client links
allLinks, err := linkService.GetAllLinks()
```

### JSON Service
```go
// Generate JSON configuration
json, err := jsonService.GetJson(clientName, host)

// Get full configuration
config, err := jsonService.GetConfig(clientName)
```

## Best Practices

### 1. Error Handling
Always check and handle errors appropriately:
```go
result, err := service.Method()
if err != nil {
    logger.Error("Operation failed:", err)
    return err
}
```

### 2. Database Transactions
Use transactions for multiple related operations:
```go
tx := database.GetDB().Begin()
defer func() {
    if r := recover(); r != nil {
        tx.Rollback()
    }
}()

// Perform operations
if err := tx.Error; err != nil {
    tx.Rollback()
    return err
}

tx.Commit()
```

### 3. Resource Management
Always clean up resources:
```go
// Close connections
defer conn.Close()

// Cancel contexts
ctx, cancel := context.WithCancel(context.Background())
defer cancel()
```

### 4. Logging
Use appropriate log levels:
- **Debug**: Detailed information for debugging
- **Info**: General informational messages
- **Warning**: Warning messages for potential issues
- **Error**: Error messages for failures

### 5. Configuration Updates
When updating configurations:
1. Validate the new configuration
2. Save to database
3. Apply changes to running core
4. Handle rollback on failure

## Extension Points

### Adding New Protocols
1. Implement protocol handler in `core` package
2. Add link generation in `util/genLink.go`
3. Add link parsing in `util/linkToJson.go`
4. Update subscription service
5. Add UI components (in frontend)

### Adding New Services
1. Create service struct in `service` directory
2. Implement service methods
3. Register in `api/apiService.go`
4. Add API endpoints in `api/apiHandler.go`
5. Update documentation

### Adding New Settings
1. Add setting key in `service/setting.go`
2. Implement getter/setter methods
3. Add default value
4. Update UI settings page
5. Document the new setting