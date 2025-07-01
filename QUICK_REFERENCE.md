# S-UI Quick Reference Guide

## Common Operations

### Authentication
```bash
# Login
curl -X POST http://localhost:2095/app/api/login -d "user=admin&pass=admin" -c cookies.txt

# Use authenticated requests
curl http://localhost:2095/app/api/load -b cookies.txt
```

### Client Management
```bash
# List clients
curl http://localhost:2095/app/api/clients -b cookies.txt

# Add client
curl -X POST http://localhost:2095/app/api/save \
  -d 'object=client&action=add&data=[{"name":"user1","enable":true,"password":"pass123","limitIp":3,"totalGB":10737418240,"expiryTime":1735689600}]' \
  -b cookies.txt

# Get client subscription
curl http://localhost:2096/sub/user1/link
```

### System Management
```bash
# Get system status
curl http://localhost:2095/app/api/status?r=all -b cookies.txt

# Restart Sing-Box
curl -X POST http://localhost:2095/app/api/restartSb -b cookies.txt

# Get logs
curl "http://localhost:2095/app/api/logs?c=50&l=info" -b cookies.txt

# Backup database
curl http://localhost:2095/app/api/getdb -b cookies.txt -o backup.db
```

## API Response Format

### Success
```json
{
  "success": true,
  "msg": "",
  "obj": { /* data */ }
}
```

### Error
```json
{
  "success": false,
  "msg": "error message",
  "obj": null
}
```

## Common API Parameters

### Save Operations
- `object`: Type of object (client, inbound, outbound, endpoint, tls, config, setting, user)
- `action`: Operation (add, update, del)
- `data`: JSON data for the operation

### Query Parameters
- `id`: Object identifier
- `limit`: Result limit
- `tag`: Filter by tag
- `resource`: Resource type for statistics

## Protocol Link Formats

### VLESS
```
vless://uuid@host:port?security=tls&type=tcp&sni=example.com#name
```

### VMess
```
vmess://base64({"v":"2","ps":"name","add":"host","port":port,"id":"uuid","aid":"0","net":"tcp","type":"none","tls":"tls"})
```

### Trojan
```
trojan://password@host:port?security=tls&type=tcp&sni=example.com#name
```

### Shadowsocks
```
ss://base64(method:password)@host:port#name
```

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `SUI_LOG_LEVEL` | `info` | Log level: debug, info, warn, error |
| `SUI_DEBUG` | `false` | Enable debug mode |
| `SUI_BIN_FOLDER` | `bin` | Binary folder path |
| `SUI_DB_FOLDER` | `db` | Database folder path |

## Default Ports

- Web Panel: 2095
- Subscription: 2096
- API v1: 2095/app/api/
- API v2: 2095/app/apiv2/

## File Locations

- Database: `/usr/local/s-ui/db/s-ui.db`
- Logs: System output
- Config: Stored in database
- Certificates: User-specified paths

## Troubleshooting

### Core Not Starting
1. Check logs: `curl "http://localhost:2095/app/api/logs?c=100&l=debug" -b cookies.txt`
2. Verify configuration syntax
3. Check port conflicts
4. Ensure sufficient permissions

### Authentication Issues
1. Clear cookies
2. Check session timeout settings
3. Verify credentials
4. Check domain restrictions

### Client Connection Issues
1. Verify client is enabled
2. Check expiry date
3. Monitor traffic usage
4. Verify IP limits

## Security Checklist

- [ ] Change default admin password
- [ ] Configure SSL certificates
- [ ] Set domain restrictions
- [ ] Enable session timeout
- [ ] Regular database backups
- [ ] Monitor access logs
- [ ] Use API tokens for automation
- [ ] Restrict network access