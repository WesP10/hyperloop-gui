# Implementation TODO

## Phase 1: Core Infrastructure (In Progress)
- [x] Create folder structure
- [x] Setup .gitignore and .env.example
- [ ] Implement config.py with Pydantic BaseSettings
- [ ] Setup logging_config.py with JSON formatter

## Phase 2: USB and Serial Management
- [ ] Implement usb_port_mapper.py
  - [ ] arduino-cli subprocess integration
  - [ ] pyserial fallback detection
  - [ ] Hotplug monitoring loop
  - [ ] Stable port ID generation
  - [ ] Persistence to disk
- [ ] Implement serial_manager.py
  - [ ] Connection dataclass
  - [ ] Connection pool management
  - [ ] Explicit connection method
  - [ ] 3-attempt retry logic
  - [ ] Continuous reading loop
  - [ ] Task queue processing

## Phase 3: WebSocket Communication
- [ ] Implement buffer_manager.py
  - [ ] 10MB circular buffer
  - [ ] 80% threshold warning
  - [ ] Drop oldest policy
- [ ] Implement hub_agent.py
  - [ ] WebSocket connection management
  - [ ] Handshake authentication
  - [ ] Message routing
  - [ ] Reconnection with backoff

## Phase 4: Command Processing
- [ ] Implement command_handler.py
  - [ ] CommandEnvelope validation
  - [ ] Connection command handling
  - [ ] Task factory dispatch
- [ ] Implement tasks/
  - [ ] base.py with Task ABC
  - [ ] serial_write_task.py
  - [ ] flash_task.py with artifact download
  - [ ] restart_task.py with DTR toggle
- [ ] Implement health_reporter.py
  - [ ] psutil integration
  - [ ] Comprehensive metrics
  - [ ] 30s periodic reporting

## Phase 5: HTTP API
- [ ] Implement main.py
  - [ ] FastAPI app setup
  - [ ] Lifespan context manager
  - [ ] Component initialization
- [ ] Implement api/routes/
  - [ ] health.py
  - [ ] ports.py
  - [ ] connections.py
  - [ ] tasks.py
- [ ] Implement api/models.py with Pydantic schemas
- [ ] Implement api/middleware.py for logging

## Phase 6: Testing
- [ ] tests/conftest.py with fixtures
- [ ] tests/test_usb_mapper.py
- [ ] tests/test_serial_manager.py
- [ ] tests/test_buffer_manager.py
- [ ] tests/test_hub_agent.py
- [ ] tests/test_command_handler.py
- [ ] tests/test_health_reporter.py
- [ ] tests/test_integration.py

## Phase 7: Documentation
- [ ] Complete README.md
  - [ ] Prerequisites (arduino-cli)
  - [ ] Installation
  - [ ] Configuration
  - [ ] Running locally
  - [ ] Deployment to rpi-bridge-01
  - [ ] API documentation
  - [ ] WebSocket protocol
  - [ ] JSON log format
  - [ ] Troubleshooting

## Phase 8: Deployment Verification
- [ ] Test on rpi-bridge-01
- [ ] Verify Arduino detection
- [ ] Verify serial data flow to laptop
- [ ] Verify all logging points
