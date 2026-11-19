# Light - Daylight Brightness Boost

[![Open your Home Assistant instance and show the blueprint import dialog with this blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FSoundOfMind%2Fhome-automation-resources%2Fblob%2Fmain%2Fprojects%2Fdaylight-brightness-boost%2Fblueprints%2Flight_daylight_brightness_boost.yaml)

A Home Assistant automation blueprint for Inovelli dimmers integrated through Zigbee2MQTT, with matching device profile examples and configuration notes.

With the light off, a single UP paddle press turns the light on at the configured initial brightness. After a short verification delay, the automation checks that the light is still on and remains within the expected brightness tolerance. If the selected daylight condition is satisfied, it smoothly raises the brightness.

## Project contents

```text
daylight-brightness-boost/
├── README.md
├── blueprints/
│   └── light_daylight_brightness_boost.yaml
└── profiles/
    └── inovelli/
        ├── Inovelli VZM31-SN Dimmer Example.yaml
        ├── Inovelli VZM35-SN 3 Speed Fan Control Example.yaml
        └── Inovelli Examples Profile Notes.txt
```

## Inovelli profiles

The examples cover two Zigbee2MQTT devices:

- **VZM31-SN** — Inovelli Blue Series dimmer.
- **VZM35-SN** — Inovelli Blue Series 3-speed fan control.

The profile YAML files use the Configuration Engine profile structure (`vendor`, `model`, and `values`). They are examples and reference configurations, not universal recommendations for every installation.

The accompanying notes describe the intended settings and device-specific behavior. These profiles have been modified and documented specifically for use with this automation project; they should not be treated as generic or factory-default configurations.

### Relationship between the profile and blueprint

The VZM31-SN profile documents the dimmer configuration used during development and testing. The blueprint handles the Home Assistant automation behavior. The profile and blueprint serve different purposes:

- The profile describes device configuration.
- The blueprint monitors and controls the Home Assistant light entity and listens for the corresponding Zigbee2MQTT paddle action.

Use the profile as a reference when configuring the dimmer, then select the matching Home Assistant light and Zigbee2MQTT action topic when creating the automation.

The VZM35-SN profile is an example configuration for a conventional AC-powered three-speed fan. It is included as a related Inovelli device profile, not because the daylight brightness blueprint controls the fan.

## Requirements

- Home Assistant automation blueprint support.
- An Inovelli dimmer integrated through Zigbee2MQTT.
- A Home Assistant `light` entity for the dimmer.
- The Zigbee2MQTT action topic for that dimmer.
- The Home Assistant Sun integration for either sun-based daylight mode.

## Importing the blueprint

In Home Assistant:

1. Go to **Settings → Automations & scenes → Blueprints**.
2. Select **Import blueprint**.
3. Paste the blueprint's GitHub URL:

   `https://github.com/SoundOfMind/home-automation-resources/blob/main/projects/daylight-brightness-boost/blueprints/light_daylight_brightness_boost.yaml`

4. Preview the blueprint and select **Import blueprint**.
5. Create an automation from the imported blueprint and configure its inputs.

Home Assistant remembers the import URL as the blueprint source, allowing you to use **Re-import blueprint** when an updated version is published.

## Inputs and defaults

| Input | Default | Purpose |
|---|---:|---|
| Dimmer | Required | Home Assistant light entity to monitor and control |
| MQTT action topic | `zigbee2mqtt/{device_name}/action` | Replace `{device_name}` with the Zigbee2MQTT friendly name, preserving spaces if present |
| Verification delay | 2 seconds | Wait before checking brightness |
| Expected initial brightness | 30% | Expected starting brightness |
| Brightness tolerance | ±2% | Allowed variation around the expected brightness |
| Daylight brightness | 60% | Target brightness when daylight conditions pass |
| Brightness transition | 1 second | Transition duration |
| Daylight condition | Sun above horizon | Selects the daylight test |
| Minimum sun elevation | 5° | Threshold for the sun-elevation mode |
| Time window start | 07:00 | Start of the time-window mode |
| Time window end | 19:00 | End of the time-window mode |

## MQTT action topic

Zigbee2MQTT publishes the Inovelli paddle action as a plain payload on a topic ending in `/action`. The blueprint listens for `up_single`.

The input defaults to a placeholder topic:

```text
zigbee2mqtt/{device_name}/action
```

Replace `{device_name}` with the device's Zigbee2MQTT friendly name. For example, if the friendly name is `Bed1 Ceiling`, enter:

```text
zigbee2mqtt/Bed1 Ceiling/action
```

Use the Zigbee2MQTT friendly name, not the Home Assistant entity ID. The placeholder is instructional text; Home Assistant does not substitute it automatically.

## Trigger and verification behavior

The blueprint has two independent triggers:

- MQTT payload `up_single` on the configured action topic.
- The selected light turning on.

Either trigger starts the verification sequence. They are alternatives; both events do not need to occur. After the delay, the automation checks the light state and brightness, then checks the selected daylight condition. If all checks pass, it calls `light.turn_on` with the configured target brightness and transition.

The brightness check converts Home Assistant's 0–255 brightness attribute to a percentage before comparing it with the configured starting brightness and tolerance.

## Daylight modes

- **Sun above horizon:** requires `sun.sun` to be `above_horizon`.
- **Minimum sun elevation:** requires the Sun integration's elevation to meet the configured threshold. Negative values can permit the boost before sunrise or after sunset.
- **Time window:** permits the boost during the configured daily interval, including intervals that cross midnight.

## Multiple dimmers

Create a separate automation instance for each dimmer. Each instance has its own light entity, MQTT action topic, and settings.

## Configuration Engine

Configuration Engine is a separate application for inspecting, capturing, comparing, and applying device profiles through Zigbee2MQTT. This repository keeps device examples and related Home Assistant blueprints organized by project, independently of the Configuration Engine application.

Repository: https://github.com/SoundOfMind/configuration_engine
