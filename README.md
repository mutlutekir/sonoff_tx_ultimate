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
alias: Sonoff TX Ultimate - Kontrol Paneli
description: "TX Ultimate cihazınıza görsel bir arayüzle renk, parlaklık ve efekt komutları gönderin."
icon: mdi:lightbulb-multiple
fields:
  target_device:
    name: "Sonoff Cihazı"
    description: "Listeden komut göndermek istediğiniz cihazı seçin."
    required: true
    selector:
      device:
        integration: sonoff
  rgb_color:
    name: "Işık Rengi"
    description: "Halo ışığı için renk paletinden seçim yapın."
    default: [255, 0, 0]
    selector:
      color_rgb: {}
  brightness:
    name: "Parlaklık"
    description: "Işık şiddetini ayarlayın."
    default: 100
    selector:
      number:
        min: 0
        max: 100
        unit_of_measurement: "%"
  light_effect:
    name: "Işık Efekti"
    description: "Cihazın oynatacağı görsel animasyon."
    default: "0"
    selector:
      select:
        options:
          - label: "0 - Efekt Kapalı"
            value: "0"
          - label: "1 - Dalga Efekti"
            value: "1"
          - label: "2 - Nefes Efekti"
            value: "2"
          - label: "3 - Döngü Efekti"
            value: "3"
          - label: "4 - Hızlı Geçiş"
            value: "4"
          - label: "5 - Renk Patlaması"
            value: "5"
  sound_effect:
    name: "Ses Efekti"
    description: "Beraberinde çalacak ses efekti."
    default: "0"
    selector:
      select:
        options:
          - label: "0 - Ses Kapalı"
            value: "0"
          - label: "1 - Bip"
            value: "1"
          - label: "2 - Çift Bip"
            value: "2"
          - label: "3 - Melodi 1"
            value: "3"
          - label: "4 - Alarm Zili"
            value: "4"
          - label: "5 - Bildirim Sesi"
            value: "5"
  volume:
    name: "Ses Seviyesi"
    description: "Çalınacak efektin ses düzeyi."
    default: 80
    selector:
      number:
        min: 0
        max: 100
        unit_of_measurement: "%"
  top_light:
    name: "Üst Işık Paneli"
    description: "Üst durum ışıkları aktif edilsin mi?"
    default: true
    selector:
      boolean: {}
  bottom_light:
    name: "Alt Işık Paneli"
    description: "Alt durum ışıkları aktif edilsin mi?"
    default: true
    selector:
      boolean: {}
sequence:
  - variables:
      ewelink_id: >-
        {% set idents = device_attr(target_device, 'identifiers') | list %}
        {% set ns = namespace(id=target_device) %}
        {% for ident in idents %}
          {% if ident[0] == 'sonoff' %}
            {% set ns.id = ident[1] %}
          {% endif %}
        {% endfor %}
        {{ ns.id }}
  - service: sonoff.send_command
    data:
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
```
