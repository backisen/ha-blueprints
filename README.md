# Home Assistant Blueprints

A collection of reusable Home Assistant automation blueprints for monitoring, alerting, and lighting control.

## Blueprints

### temp_deviation_watchdog.yaml

**Temperaturavvikelse – larm & påminnelse**

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fbackisen%2Fha-blueprints%2Fblob%2Fmaster%2Ftemp_deviation_watchdog.yaml)

Monitors the difference between a current temperature sensor and a target temperature entity. Alerts on deviation and sends reminders until the temperature is back within tolerance.

**Features:**
- Separate high/low deviation thresholds (set to 0 to disable a direction)
- Second critical threshold tier with separate iOS notification level
- Optional connectivity sensor to prevent false alarms when the device is offline
- Configurable trigger duration and reminder interval
- Persistent notification in WebUI
- Recovery notification when temperature returns to normal

### offline_watchdog.yaml

**Offline Watchdog – enhetsövervakning**

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fbackisen%2Fha-blueprints%2Fblob%2Fmaster%2Foffline_watchdog.yaml)

Monitors a binary sensor for connectivity and sends notifications when a device goes offline. Sends reminders at a configurable interval and optionally notifies when the device comes back online.

**Features:**
- Configurable offline threshold and reminder interval
- Push notifications with configurable iOS interruption level
- Persistent notification in WebUI
- Online recovery notification with offline duration
- Push notifications use tags to replace (not stack) previous notifications

### low_battery_watchdog.yaml

**Low Battery Watchdog – batteriövervakning**

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fbackisen%2Fha-blueprints%2Fblob%2Fmaster%2Flow_battery_watchdog.yaml)

Scans all battery sensors and creates notifications when battery levels are low. Supports two severity tiers with separate notification targets.

**Features:**
- Configurable warning and critical battery thresholds
- Quiet hours to suppress notifications at night
- Entity exclusion list
- Separate notify services for warning vs critical level
- Critical alerts use iOS critical push sound
- Persistent notification in WebUI

### unavailable_entity_watchdog.yaml

**Unavailable Entity Watchdog – otillgängliga entiteter**

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fbackisen%2Fha-blueprints%2Fblob%2Fmaster%2Funavailable_entity_watchdog.yaml)

Scans all entities and reports those with state `unavailable` or `unknown`. Filters by domain and individual entity exclusion lists.

**Features:**
- Configurable check interval (hours) and run-on-start option
- Quiet hours to suppress notifications at night
- Domain exclusion list (comma-separated, sensible defaults for helper domains)
- Individual entity exclusion list
- Persistent notification in WebUI (auto-dismissed when all clear)
- Optional push notifications via configurable notify services

### motion_light.yaml

**Rörelsestyrd belysning**

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fbackisen%2Fha-blueprints%2Fblob%2Fmaster%2Fmotion_light.yaml)

Turns on lights when motion is detected and turns them off after a configurable delay. Supports multiple motion sensors and flexible light targeting. Uses `mode: restart` so that new motion resets the off-timer.

**Features:**
- Multiple motion sensors (select any number)
- Flexible light target (entities, areas, or devices)
- Configurable delay before turning off (10–3600 seconds, slider)
- Optional brightness and separate fade-in / fade-out transition times
- Optional illuminance sensor condition (don't turn on if already bright enough)
- Optional sun elevation condition (only activate when dark enough)
- Optional blocker entity (e.g. an input_boolean to disable during movie night)

## Installation

Click the **Import Blueprint** button above for the blueprint you want, or manually:

1. Copy the desired `.yaml` file(s) to your Home Assistant `blueprints/automation/custom/` directory.
2. Reload automations or restart Home Assistant.
3. Create a new automation and select the blueprint from the list.

## License

MIT
