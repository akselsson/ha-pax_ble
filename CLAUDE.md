# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Home Assistant custom integration (`pax_ble`) for controlling Pax/Vent-Axia ventilation fans via Bluetooth Low Energy. Supports three device models: Calima, Svara, and Svensa.

## Local Home Assistant Instance

A local HA instance is available for testing:
- **URL**: http://localhost:8123
- **Installation directory**: `/Users/patrikakselsson/Projects/Pax/ha_local`
- **Config directory**: `/Users/patrikakselsson/Projects/Pax/ha_local/homeassistant_config`
- **Integration install**: The `pax_ble` integration is symlinked into `custom_components/` so changes are reflected immediately. Restart HA to pick up changes.

## Validation Commands

```bash
# No local build or test suite. Validation runs via GitHub Actions:
# - hassfest: validates manifest.json and integration structure
# - HACS validation: validates for Home Assistant Community Store compatibility
```

## Architecture

Three-layer design: **Device → Coordinator → Entity**

- **Device layer** (`devices/`): BLE connection management and GATT characteristic read/write. `BaseDevice` handles connection lifecycle; `Calima` and `Svensa` subclasses implement model-specific GATT operations using `struct` for binary packing/unpacking. PIN-based authorization is required before writes.

- **Coordinator layer** (`coordinator.py`, `coordinator_calima.py`, `coordinator_svensa.py`): Extends Home Assistant's `DataUpdateCoordinator`. Manages polling intervals (normal: 300s, fast: 5s after writes for 10 reads), maintains internal `_state` dict, and handles connection failure tracking (max 5 attempts). Use `helpers.py:getCoordinator()` factory to create the right coordinator per device model.

- **Entity layer** (`sensor.py`, `switch.py`, `number.py`, `select.py`, `time.py`): Each file defines entities for one HA platform. All entities inherit from `PaxCalimaEntity` (in `entity.py`) which reads from the coordinator's state dict. Entities are model-aware — many are conditionally created based on `DeviceModel`.

- **Config flow** (`config_flow.py`): Supports automatic Bluetooth discovery (Calima only) and manual device addition. Options flow allows editing/removing devices. Multi-device support.

## Key Conventions

- Python 3.10+ required (`match/case` statements used for device-specific logic)
- Named tuples for structured device data (FanState, BoostMode, etc.)
- Entity unique IDs follow pattern: `{device_id}-{entity_name}`
- Connection uses `bleak-retry-connector` with disconnect callbacks and exponential backoff
- Translations in `translations/` (en, sv, nb, fi)
- Single registered service: `request_update` (forces immediate data refresh)
