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

### ambient_light_schedule.yaml

**Ambient belysning – sol-elevation + vardag/helg-schema**

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fbackisen%2Fha-blueprints%2Fblob%2Fmaster%2Fambient_light_schedule.yaml)

Controls ambient lighting based on sun elevation and configurable morning/evening schedules with weekday/weekend awareness via the workday sensor. Correctly restores state on HA restart.

**Features:**
- Sun elevation triggers (configurable thresholds for on/off)
- Separate weekday and weekend/holiday schedules via `input_datetime` helpers
- Workday sensor awareness (adapts to holidays)
- Optional scene support for turn-on and turn-off
- Optional per-light brightness map via a `variable` sensor
- Correct state restoration on HA restart (with 30 s startup delay)
- Flexible light targeting (entities, areas, or devices)

#### Per-light brightness map (optional)

Instead of turning on all lights at their last-used brightness, you can configure a brightness map that ensures each light always turns on at a specific brightness level. This prevents manually adjusted dimmers from "sticking" at the wrong brightness the next evening.

**Prerequisites:** Install the [hass-variables](https://github.com/enkama/hass-variables) integration via HACS.

**1. Create the variable sensor**

In Home Assistant, go to **Settings > Devices & Services > Add Integration** and search for **Variables+History**. Choose **Add Sensor** and configure:

| Field | Value |
|---|---|
| Variable ID | `ambient_brightness_map` |
| Name | Ambient brightness-karta |
| Icon | `mdi:brightness-percent` |
| Restore | Yes |
| Exclude from recorder | Yes (recommended) |

On the second page, set the initial value to `active` and add your lights as attributes.

**Important:** Attribute keys use the entity ID **without** the `light.` prefix (HA interprets dots in attribute keys as nesting, so we strip the domain prefix):

```json
{
  "kitchen_island_strip": 100,
  "living_room_downlight": 80,
  "bedroom_window": 60
}
```

Each key is the light entity ID minus the `light.` prefix, and each value is brightness in percent (0–100).

**2. Reference the sensor in your automation**

When creating an automation from this blueprint, select your variable sensor in the **Brightness-karta** input field:

```yaml
use_blueprint:
  path: custom/ambient_light_schedule.yaml
  input:
    light_target:
      entity_id: light.my_ambient_group
    brightness_map: sensor.ambient_brightness_map
    # ... other inputs
```

**3. Update brightness values**

Call the `variable.update_sensor` service to change brightness for individual lights. Attributes are merged, so you only need to specify the lights you want to change:

```yaml
action: variable.update_sensor
target:
  entity_id: sensor.ambient_brightness_map
data:
  attributes:
    kitchen_island_strip: 60
    living_room_downlight: 40
```

The new values take effect the next time the automation turns on the lights.

**4. Snapshot current brightness (optional script)**

Instead of manually entering brightness values, you can create a script that automatically discovers all light groups ending in `_ambient`, reads each member's current brightness, and saves it to the map. New lights added to a group are picked up automatically.

Adjust your lights to the desired levels, then run the script:

```yaml
# scripts.yaml
ambient_brightness_snapshot:
  alias: "Ambient – Spara aktuell brightness"
  description: >
    Discovers all *_ambient light groups, reads current brightness
    from their members, and updates the variable sensor. Lights
    that are off or lack a brightness attribute keep their existing
    value. New lights added to a group are included automatically.
  icon: mdi:content-save-outline
  sequence:
    - variables:
        lights_to_update: >
          {% set ns = namespace(items=[], seen=[]) %}
          {% set ambient_groups = states.light
             | selectattr('entity_id', 'search', '_ambient$')
             | map(attribute='entity_id') | list %}
          {% for group in ambient_groups %}
            {% set members = state_attr(group, 'entity_id') %}
            {% if members %}
              {% for member in members %}
                {% if member not in ns.seen %}
                  {% set ns.seen = ns.seen + [member] %}
                  {% set key = member | replace('light.', '', 1) %}
                  {% set br = state_attr(member, 'brightness') %}
                  {% if br is not none %}
                    {% set pct = (br / 255 * 100) | round | int %}
                    {% set ns.items = ns.items + [{'key': key, 'pct': pct}] %}
                  {% endif %}
                {% endif %}
              {% endfor %}
            {% endif %}
          {% endfor %}
          {{ ns.items }}
    - condition: template
      value_template: "{{ lights_to_update | length > 0 }}"
    - repeat:
        for_each: "{{ lights_to_update }}"
        sequence:
          - action: variable.update_sensor
            target:
              entity_id: sensor.ambient_brightness_map
            data:
              attributes:
                "{{ repeat.item.key }}": "{{ repeat.item.pct }}"
```

Call via **Developer Tools > Services** → `script.ambient_brightness_snapshot`, or add a button to your dashboard.

**Behavior:**
- Scene input takes priority over brightness map (if both are set, the scene is used)
- If a light is not found in the brightness map, it defaults to 100%
- If the brightness map sensor is unavailable, the blueprint falls back to simple `light.turn_on`
- The brightness map input is optional — without it, the blueprint behaves exactly as before

---

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
