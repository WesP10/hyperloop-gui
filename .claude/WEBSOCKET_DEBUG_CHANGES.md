# WebSocket Connection Debugging - Changes Summary

## Problem Diagnosed

The RPI service was failing to connect to the cloud service with error:
```
[Errno 111] Connection refused
```

**Root Cause:** The RPI was using the default `SERVER_ENDPOINT=ws://localhost:8080/hub`, which only works if both services run on the same machine. For RPI deployment, it needs the actual IP address or hostname of the cloud service.

## Changes Made

### 1. Enhanced Logging in `rpi-hub-service`

#### File: `src/hub_agent.py`
- Added detailed connection logging with endpoint parsing (scheme, host, port, path)
- Added DNS resolution check and logging before connection attempt
- Enhanced error messages with specific guidance for common errors:
  - Connection refused → "Check if cloud service is running and accessible"
  - DNS resolution failed → "Check hostname in SERVER_ENDPOINT"
  - Network unreachable → "Check network connection"
  - Connection timeout → "Check firewall settings"
- Added errno logging for better debugging
- Logs now show resolved IP addresses for non-localhost connections

**Example Enhanced Logs:**
```json
{"event": "ws_connecting", "endpoint": "ws://192.168.1.100:8080/hub", "host": "192.168.1.100", "port": 8080}
{"event": "dns_resolved", "hostname": "myserver.com", "ip": "203.0.113.50"}
{"event": "ws_attempting_connection"}
{"event": "ws_connection_error", "error": "...", "errno": 111}
```

#### File: `src/main.py`
- Added startup logging to show:
  - Hub ID being used
  - Server endpoint configured
  - Whether device token is set (shows length but not the token itself)
- Added warnings for common misconfigurations:
  - Warning if using default localhost endpoint
  - Warning if DEVICE_TOKEN is not set

**Example Startup Logs:**
```json
{"message": "Starting RPi Hub Service", "hub_id": "rpi-bridge-01", "server_endpoint": "ws://localhost:8080/hub", "device_token_set": true}
{"level": "WARNING", "message": "Using default localhost endpoint - ensure cloud service is accessible"}
```

### 2. Enhanced Logging in `cloud-services`

#### File: `src/websocket/hub_endpoint.py`
- Added client host/port logging when WebSocket connections are attempted
- Shows where connections are coming from for security and debugging

**Example Logs:**
```
INFO: WebSocket connection attempt from 192.168.1.50:54321
INFO: WebSocket connection accepted from 192.168.1.50:54321
```

#### File: `src/main.py`
- Added startup logging showing:
  - Listen address and port
  - WebSocket endpoint URL
  - Valid device tokens (hub IDs only, not the actual tokens)
  - Environment (dev/prod)

**Example Startup Logs:**
```
INFO: Starting Cloud Service on 0.0.0.0:8080
INFO: WebSocket endpoint: ws://0.0.0.0:8080/hub
INFO: Valid device tokens: ['rpi-bridge-01', 'rpi-bridge-02']
INFO: Environment: development
```

### 3. Documentation

#### New File: `rpi-hub-service/QUICKFIX.md`
- 2-minute quick reference for fixing connection errors
- Step-by-step instructions with exact commands
- Network testing commands
- Common values reference table

#### New File: `rpi-hub-service/RPI_SETUP.md`
- Comprehensive setup guide
- Detailed explanation of each configuration option
- Multiple deployment scenarios (dev, production, multi-RPI)
- Troubleshooting section with specific error codes
- Example configurations for common setups

#### Updated: `rpi-hub-service/README.md`
- Added prominent links to QUICKFIX.md and RPI_SETUP.md
- Updated configuration section with clearer instructions
- Added warning about replacing placeholder values

## What to Do Next

### If the Cloud Service is on Your PC:

1. **On your PC**, find your IP address:
   ```powershell
   ipconfig
   # Look for IPv4 Address, e.g., 192.168.1.100
   ```

2. **On the RPI**, create or update `.env`:
   ```bash
   cd /path/to/rpi-hub-service
   nano .env
   ```
   
   Add:
   ```dotenv
   HUB_ID=rpi-bridge-01
   SERVER_ENDPOINT=ws://192.168.1.100:8080/hub
   DEVICE_TOKEN=dev-token-rpi-bridge-01
   ```

3. **On your PC**, ensure cloud service allows external connections:
   - Check `cloud-services/.env` has `HOST=0.0.0.0`
   - Check firewall allows port 8080

4. **Restart both services** and check logs

### Testing the Fix

1. **From the RPI**, test connectivity:
   ```bash
   curl http://192.168.1.100:8080/health
   ```
   Should return: `{"status": "healthy"}`

2. **Start the RPI service** and watch the logs:
   ```bash
   python -m src.main
   ```

3. **Look for these log events** (in order):
   - `ws_connecting` - Shows the endpoint
   - `dns_resolved` (if not localhost) - Shows resolved IP
   - `ws_attempting_connection` - Connection attempt
   - `ws_connected` - Success!

### If Still Having Issues

The enhanced logging will now show:
- Exact endpoint being used
- DNS resolution results (if applicable)
- Specific error with helpful guidance
- Client connection attempts on cloud service side

Check both service logs to identify where the connection is failing.

## Files Changed

- ✅ `rpi-hub-service/src/hub_agent.py` - Enhanced connection logging
- ✅ `rpi-hub-service/src/main.py` - Startup configuration logging
- ✅ `cloud-services/src/websocket/hub_endpoint.py` - Client logging
- ✅ `cloud-services/src/main.py` - Startup information
- ✅ `rpi-hub-service/QUICKFIX.md` - New quick reference
- ✅ `rpi-hub-service/RPI_SETUP.md` - New comprehensive guide
- ✅ `rpi-hub-service/README.md` - Updated with links to guides

## How to Deploy

### Push Changes to Git
```bash
cd rpi-hub-service
git add .
git commit -m "Add enhanced WebSocket connection logging and setup documentation"
git push

cd ../cloud-services
git add .
git commit -m "Add startup logging and client connection details"
git push
```

### Update on RPI
```bash
ssh pi@rpi-bridge-01
cd /path/to/rpi-hub-service
git pull
# Create .env file with correct SERVER_ENDPOINT (see QUICKFIX.md)
# Restart service
```

### Update Cloud Service
```bash
cd /path/to/cloud-services
git pull
# Ensure .env has HOST=0.0.0.0
# Restart service
```
