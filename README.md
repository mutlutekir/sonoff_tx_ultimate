# Sonoff TX Ultimate - Visual Control Panel (Home Assistant)

A Home Assistant script that provides a fully visual, interactive control panel for your **Sonoff TX Ultimate** smart touch switches. 

Control the RGB halo lights, sound effects, and status LEDs directly from your Home Assistant dashboard—**without installing ESPHome, changing firmware, or voiding your warranty.** By utilizing local JSON commands through the SonoffLAN integration, this script unlocks the almost-full potential of your device using a clean, user-friendly UI.

**Developer:** Mutlu Tekir | **YouTube:** [@mutlutekir](https://youtube.com/@mutlutekir)  
*(If you feature this method in your own projects, blogs, or videos, a shoutout or a link to my YouTube channel would be greatly appreciated. Knowledge grows when shared!)*

---

## ✨ Features & UI Controls

When you add this script to your dashboard or call it in an automation, it provides the following intuitive controls:

*   **🎯 Sonoff Device:** A smart dropdown that filters your Home Assistant entities and only lists your integrated Sonoff devices (e.g., *Entryway, Hallway*). Just click and select!
*   **🎨 Light Color:** A native RGB color picker to instantly change the ambient halo light.
*   **🔆 Brightness:** A slider to adjust the light intensity from 0% to 100%.
*   **✨ Light Effect:** A dropdown menu to trigger the device's built-in visual animations (e.g., *5 - Color Burst*).
*   **🔊 Sound Effect:** A dropdown menu for the accompanying sound (e.g., *0 - Sound Off*, or various chimes/beeps).
*   **🔉 Volume Level:** A slider to set the volume of the selected sound effect (0% to 100%).
*   **↕️ Top & Bottom Light Panels:** Simple toggle switches to activate or deactivate the top and bottom status indicator LEDs.

> **Note on "Response Variable":** If you see this field at the bottom of the interface, it is a standard Home Assistant feature for scripts. Since this script only sends commands and doesn't read data back, you can safely leave this field empty.

---

## 📋 Prerequisites

1. **Home Assistant** (Up and running).
2. **SonoffLAN Integration** by [AlexxIT](https://github.com/AlexxIT/SonoffLAN) (Easily installable via HACS).
3. At least one **Sonoff TX Ultimate** switch connected via the SonoffLAN integration.

---

## 🚀 Installation

You don't need to mess with complex file systems. You can create this script directly from your Home Assistant UI:

1. Go to **Settings > Automations & Scenes > Scripts**.
2. Click **Add Script** -> **Create New Script**.
3. Click the three dots (⋮) in the top right corner and select **Edit in YAML**.
4. Delete any default text, paste the code below, and click **Save**:

<details>
<summary><b>Click to expand the Script YAML code</b></summary>
```yaml
alias: Sonoff TX Ultimate - Control Panel
description: >-
  Send color, brightness, and effect commands to your TX Ultimate device using a visual interface.
icon: mdi:lightbulb-multiple
fields:
  target_device:
    name: Sonoff Device
    description: Select the device you want to send commands to from the list.
    required: true
    selector:
      device:
        integration: sonoff
  rgb_color:
    name: Light Color
    description: Select a color from the palette for the halo light.
    default:
      - 255
      - 0
      - 0
    selector:
      color_rgb: {}
  brightness:
    name: Brightness
    description: Adjust the light intensity.
    default: 100
    selector:
      number:
        min: 0
        max: 100
        unit_of_measurement: "%"
  light_effect:
    name: Light Effect
    description: The visual animation the device will play.
    default: "0"
    selector:
      select:
        options:
          - label: 0 - Effect Off
            value: "0"
          - label: 1 - Wave Effect
            value: "1"
          - label: 2 - Breath Effect
            value: "2"
          - label: 3 - Cycle Effect
            value: "3"
          - label: 4 - Fast Transition
            value: "4"
          - label: 5 - Color Burst
            value: "5"
  sound_effect:
    name: Sound Effect
    description: The accompanying sound effect to play.
    default: "0"
    selector:
      select:
        options:
          - label: 0 - Sound Off
            value: "0"
          - label: 1 - Beep
            value: "1"
          - label: 2 - Double Beep
            value: "2"
          - label: 3 - Melody 1
            value: "3"
          - label: 4 - Alarm Chime
            value: "4"
          - label: 5 - Notification Sound
            value: "5"
  volume:
    name: Volume Level
    description: The volume level of the effect to be played.
    default: 80
    selector:
      number:
        min: 0
        max: 100
        unit_of_measurement: "%"
  top_light:
    name: Top Light Panel
    description: Should the top status lights be activated?
    default: true
    selector:
      boolean: {}
  bottom_light:
    name: Bottom Light Panel
    description: Should the bottom status lights be activated?
    default: true
    selector:
      boolean: {}
sequence:
  - variables:
      ewelink_id: >-
        {% set idents = device_attr(target_device, 'identifiers') | list %} {%
        set ns = namespace(id=target_device) %} {% for ident in idents %}
          {% if ident[0] == 'sonoff' %}
            {% set ns.id = ident[1] %}
          {% endif %}
        {% endfor %} {{ ns.id }}
  - data:
      device: "{{ ewelink_id }}"
      preEffects:
        lightEffect: "{{ light_effect | int }}"
        soundEffect: "{{ sound_effect | int }}"
        statusLight: "on"
        statusLightTop: "{{ 1 if top_light else 0 }}"
        statusLightBelow: "{{ 1 if bottom_light else 0 }}"
        r: "{{ rgb_color[0] }}"
        g: "{{ rgb_color[1] }}"
        b: "{{ rgb_color[2] }}"
        br: "{{ brightness | int }}"
        volume: "{{ volume | int }}"
    action: sonoff.send_command
