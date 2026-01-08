# Implementation Notes

## Design Decisions

### No Auto-Connection
Serial ports are detected and monitored but NOT opened until explicit connection command received from server. This prevents:
- Unauthorized access to devices
- Resource conflicts
- Session management issues

### Arduino-CLI Integration
Primary method for baud rate detection. Falls back to default 9600 if:
- arduino-cli not installed
- Detection fails
- Device not recognized

Subprocess command: `arduino-cli board list --format json`

### Port ID Stability
Generated from USB location and serial number to maintain consistent IDs across:
- Device reconnection
- System reboots
- Port reassignment

### Buffer Strategy
10MB circular buffer with:
- FIFO drop policy (drop oldest first)
- Single warning at 80% threshold
- Single warning when dropping below 80%
- Per-message drop logging

### Retry Logic
Connection attempts: 3 tries with exponential backoff
- Attempt 1: Immediate
- Attempt 2: After 1s
- Attempt 3: After 2s
- Abandon: After 4s delay

### Artifact Management
Flash artifacts cached to `/tmp/rpi-hub-artifacts/`
- Downloads via server endpoint
- Automatic cleanup after 24 hours
- Prevents disk space issues

## Security Considerations
- Device token authentication for WebSocket
- No public API exposure (local network only)
- Serial port access requires server authorization

## Performance Targets
- Port scan: <500ms
- Serial read latency: <10ms
- WebSocket message latency: <50ms
- Health report overhead: <1% CPU

## Testing Requirements
All data flow paths must have tests:
1. Port detection → mapping
2. Connection command → serial open
3. Serial read → buffer → WebSocket
4. WebSocket command → task → serial write
5. Hotplug → detection → event
6. Buffer overflow → drop → warning
7. Connection failure → retry → abandon
