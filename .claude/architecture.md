# RPi Hub Service Architecture

## Overview
FastAPI server running on Raspberry Pi that bridges Arduino serial data to laptop over WiFi.

## Core Components

### Hub Agent
- Manages WebSocket connection to server
- Handles authentication with device token
- Routes messages between serial ports and server
- Automatic reconnection with exponential backoff

### Serial Manager
- Connection pooling for multiple serial ports
- Waits for explicit connection commands (no auto-connect)
- 3-attempt retry with exponential backoff (1s, 2s, 4s)
- Continuous reading from active connections

### USB Port Mapper
- Uses arduino-cli for baud rate detection (fallback to pyserial)
- Generates stable port IDs from USB location/serial number
- Continuous hotplug monitoring (2s scan interval)
- Persists mappings to disk

### Buffer Manager
- 10MB circular FIFO buffer for telemetry
- Drops oldest messages when full
- Warning log at 80% capacity threshold

### Command Handler
- Validates CommandEnvelope from WebSocket
- Dispatches to task factories
- Manages priority queue

### Task System
- SerialWriteTask: Send data to serial port
- FlashTask: Download artifacts and flash firmware
- RestartTask: Reset device via DTR toggle

### Health Reporter
- 30s periodic reports with comprehensive metrics:
  - CPU/memory/disk/temperature
  - Active connection count
  - Task queue size
  - Buffer utilization
  - Per-port error counts
  - Uptime seconds

## Data Flow

1. Server → WebSocket → Hub Agent → Command Handler → Task Queue
2. Task Execution → Serial Manager → Arduino
3. Arduino → Serial Manager → Buffer Manager → Hub Agent → WebSocket → Server

## Logging Strategy
All logs as JSON to stdout with structure:
```json
{
  "timestamp": "ISO8601",
  "level": "INFO|ERROR|DEBUG",
  "module": "component_name",
  "event": "event_type",
  "context": {...}
}
```

## Configuration
- Environment variables (.env)
- YAML config (config/config.yaml)
- Pydantic BaseSettings for validation
