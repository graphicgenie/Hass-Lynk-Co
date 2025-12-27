# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Lynk & Co Home Assistant custom integration for controlling European Lynk & Co vehicles. Provides climate control, door locking, engine start/stop (experimental), and 50+ sensor entities for vehicle monitoring.

**Status**: Maintenance mode - the maintainer notes Lynk may break the integration in the future.

## Development Commands

```bash
# Lint with flake8 (max line length: 140)
flake8 custom_components/lynkco/

# Validate Home Assistant integration manifest
# Runs automatically via GitHub Actions (hassfest.yml)
```

No build step required - changes are immediately testable by copying to Home Assistant's `custom_components/` directory.

## Architecture

### Core Components (`custom_components/lynkco/`)

| File | Purpose |
|------|---------|
| `__init__.py` | Entry point, DataUpdateCoordinator setup, service registration |
| `config_flow.py` | UI configuration with two auth methods (direct login + 2FA, redirect URI) |
| `token_manager.py` | JWT/CCC token handling with async refresh |
| `login_flow.py` | Authentication flows (direct, 2FA SMS, OAuth redirect) |
| `data_fetcher.py` | API calls to `/data/shadow`, `/data/record`, address geocoding |
| `remote_control_manager.py` | Vehicle command implementations (climate, doors, horn, etc.) |
| `expected_state_monitor.py` | Post-command state verification with auto-polling |

### Entity Platforms

- `sensor.py` + `sensors/*.py` - 15 sensor modules covering battery, charging, fuel, climate, position, maintenance, etc.
- `binary_sensor.py` - Climate active, vehicle running, position trusted
- `lock.py` - Door lock entity
- `device_tracker.py` - GPS location

### Key Patterns

**DataUpdateCoordinator**: Central data polling with configurable interval (1-24 hours, default 4 hours). Uses debouncer with 10-second cooldown. "Dark hours" feature skips updates during configured timeframe.

**Sensor Base Class** (`sensors/lynk_co_sensor.py`): Uses dot-notation data paths (e.g., `vehicle_shadow.battery.level`) to extract values from API responses. Supports state mapping for enum conversion.

**Entity Identification**: All entities keyed by VIN (Vehicle Identification Number).

**Service Registration**: Dynamic registration based on experimental features toggle. Services use `ExpectedStateMonitor` to verify state changes.

## API Data Sources

- `vehicle_shadow` - Real-time vehicle state
- `vehicle_record` - Historical/record data
- `vehicle_address` - Geocoded address from GPS coordinates

## Configuration

UI-based config flow only (no YAML). Options include:
- Scan interval (60-1440 minutes)
- Dark hours start/end
- Experimental features (start/stop engine)

## Dependencies

- `requests>=2.31.0` - HTTP client
- `pkce>=1.0.3` - OAuth PKCE flow
- Home Assistant 2023.1+
