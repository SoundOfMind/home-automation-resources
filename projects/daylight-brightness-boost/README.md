# Home Automation Resources

A collection of device profiles and Home Assistant automations that are designed to work together.

The repository has two related parts:

- **Profiles** document known-good device settings. They can be used as reference configurations and, where compatible, imported into Configuration Engine.
- **Blueprints** use device capabilities and configuration choices to provide repeatable Home Assistant behavior.

The intent is to keep the device configuration examples close to the automations they support, while keeping both independent of the Configuration Engine application.

## Repository layout

```text
profiles/
  inovelli/
    Inovelli VZM31-SN Dimmer Example.yaml
    Inovelli VZM35-SN 3 Speed Fan Control Example.yaml
    Inovelli Examples Profile Notes.txt

blueprints/
  automation/
    inovelli/
      light_daylight_brightness_boost.yaml
```

## Inovelli profiles

The Inovelli examples cover two Zigbee2MQTT devices:

- **VZM31-SN** — Inovelli Blue Series dimmer.
- **VZM35-SN** — Inovelli Blue Series 3-speed fan control.

The profile YAML files use the Configuration Engine profile structure (`vendor`, `model`, and `values`). They are examples and reference configurations, not universal recommendations for every installation.

The accompanying notes file describes the intended settings and device-specific behavior.

### Relationship to the daylight brightness blueprint

The `Light - Daylight Brightness Boost` blueprint is intended for an Inovelli dimmer configured so that a single UP paddle press from OFF turns the light on at the expected initial brightness. The supplied VZM31-SN example profile documents the corresponding dimming-rate settings used during testing.

The profile and blueprint serve different purposes:

- The profile describes device configuration.
- The blueprint describes the Home Assistant automation behavior.

Use the profile as a reference when configuring the dimmer, then select the matching Home Assistant light and Zigbee2MQTT action topic when creating the automation.

## Blueprint: Light - Daylight Brightness Boost

With the light off, a single UP paddle press turns the light on at the configured initial brightness. After a short verification delay, the automation checks that the light is still on and remains within the expected brightness tolerance. If the selected daylight condition is satisfied, it smoothly raises the brightness.

### Requirements

- Home Assistant automation blueprint support.
- An Inovelli dimmer integrated through Zigbee2MQTT.
- A Home Assistant `light` entity for the dimmer.
- The Zigbee2MQTT action topic for that dimmer.
- The Home Assistant Sun integration for either sun-based daylight mode.

### Installation

Copy `blueprints/light_daylight_brightness_boost.yaml` into the Home Assistant directory:

```text
config/blueprints/automation/
```

Reload automations or restart Home Assistant, then create an automation from the imported blueprint.

### Inputs and defaults

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

### MQTT action topic

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

### Trigger and verification behavior

The blueprint has two independent triggers:

- MQTT payload `up_single` on the configured action topic.
- The selected light turning on.

Either trigger starts the verification sequence. They are alternatives, not a requirement for both events to occur. After the delay, the automation checks the light state and brightness, then checks the selected daylight condition. If all checks pass, it calls `light.turn_on` with the configured target brightness and transition.

The brightness check converts Home Assistant's 0–255 brightness attribute to a percentage before comparing it with the configured starting brightness and tolerance.

### Daylight modes

- **Sun above horizon:** requires `sun.sun` to be `above_horizon`.
- **Minimum sun elevation:** requires the Sun integration's elevation to meet the configured threshold. Negative values can permit the boost before sunrise or after sunset.
- **Time window:** permits the boost during the configured daily interval, including intervals that cross midnight.

### Multiple dimmers

Create a separate automation instance for each dimmer. Each instance has its own light entity, MQTT action topic, and settings.

## Configuration Engine

Configuration Engine is a separate application for inspecting, capturing, comparing, and applying device profiles through Zigbee2MQTT. This repository keeps reusable device examples and related Home Assistant blueprints together without making them part of the Configuration Engine application itself.

Repository: https://github.com/SoundOfMind/configuration_engine
