# S-UI Documentation Index

Welcome to the S-UI comprehensive documentation. This project provides an advanced web panel built on SagerNet/Sing-Box for managing proxy configurations, traffic routing, and system monitoring.

## Documentation Structure

### 📚 [API Documentation](API_DOCUMENTATION.md)
Complete API reference including:
- REST API endpoints (v1 and v2)
- Authentication methods
- Request/response formats
- Code examples for all endpoints
- Error handling
- Security considerations

### 🔧 [Component Documentation](COMPONENT_DOCUMENTATION.md)
Detailed documentation for all components:
- Core components (Logger, Database, Network, etc.)
- Service components (Config, Client, Stats, etc.)
- Utility functions
- Best practices
- Extension points

### ⚡ [Quick Reference](QUICK_REFERENCE.md)
Quick reference guide featuring:
- Common operations
- API response formats
- Protocol link formats
- Environment variables
- Troubleshooting guide
- Security checklist

## Key Features

- **Multi-Protocol Support**: VLESS, VMess, Trojan, Shadowsocks, ShadowTLS, Hysteria, Hysteria2, Naive, TUIC
- **Advanced Routing**: PROXY Protocol, External, Transparent Proxy, SSL Certificate management
- **Client Management**: Traffic caps, expiration dates, IP limits, online monitoring
- **System Monitoring**: CPU, memory, network, and traffic statistics
- **Subscription Service**: Multiple format support (link/json)
- **Security**: HTTPS support, session/token authentication, domain restrictions

## Getting Started

### 1. Installation
```bash
bash <(curl -Ls https://raw.githubusercontent.com/alireza0/s-ui/master/install.sh)
```

### 2. Default Access
- **Web Panel**: http://localhost:2095/app/
- **API v1**: http://localhost:2095/app/api/
- **API v2**: http://localhost:2095/app/apiv2/
- **Subscription**: http://localhost:2096/sub/
- **Credentials**: admin/admin

### 3. First Steps
1. Change default password
2. Configure SSL certificates (optional)
3. Add clients
4. Configure inbounds/outbounds
5. Monitor system status

## API Overview

### Authentication

#### Session-based (v1)
```bash
curl -X POST http://localhost:2095/app/api/login \
  -d "user=admin&pass=admin" \
  -c cookies.txt
```

#### Token-based (v2)
```bash
curl http://localhost:2095/app/apiv2/load \
  -H "Token: your-api-token"
```

### Common Operations

#### Add Client
```bash
curl -X POST http://localhost:2095/app/api/save \
  -d 'object=client&action=add&data=[{"name":"user1","enable":true,"password":"pass123"}]' \
  -b cookies.txt
```

#### Get System Status
```bash
curl http://localhost:2095/app/api/status?r=all -b cookies.txt
```

#### Get Subscription
```bash
curl http://localhost:2096/sub/user1/link
```

## Development

### Project Structure
```
s-ui/
├── api/          # API handlers
├── app/          # Application core
├── core/         # Sing-Box core wrapper
├── service/      # Business logic
├── database/     # Data models
├── web/          # Web server
├── sub/          # Subscription service
├── util/         # Utilities
└── frontend/     # Vue.js frontend
```

### Building from Source
```bash
# Clone repository
git clone https://github.com/alireza0/s-ui
cd s-ui

# Initialize submodules
git submodule update --init --recursive

# Build
go build -o sui main.go
```

## Contributing

1. Fork the repository
2. Create feature branch
3. Commit changes
4. Update documentation
5. Submit pull request

## Support

- **GitHub Issues**: https://github.com/alireza0/s-ui/issues
- **Wiki**: https://github.com/alireza0/s-ui/wiki
- **API Wiki**: https://github.com/alireza0/s-ui/wiki/API-Documentation

## License

This project is licensed under GPL v3. See [LICENSE](LICENSE) for details.

---

Generated on: {current_date}
S-UI Version: Latest (check [releases](https://github.com/alireza0/s-ui/releases))