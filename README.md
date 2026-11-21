# Climate Control Display with ESPHome and LVGL
An ESPHome-based smart climate control interface for Home Assistant using the WT32-SC01 Plus display.

[![ESPHome: 2026.11](https://img.shields.io/badge/ESPHome-2026.11-blue.svg)](https://esphome.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

![Device Preview Climate Off](images/climate_off.jpg)

![Device Preview Climate On](images/climate_on.jpg)

## Demo Video

<a href="https://www.youtube.com/watch?v=l97-AFoVEJQ" target="_blank" rel="noopener noreferrer">▶ Click here to watch a short demo</a>

### 3D Print Case

[Thingiverse - WT32-SC01 PLUS wall enclosure](https://www.thingiverse.com/thing:6683758)

![Wall Mount](images/wall_mount.jpg)

![esp32-s3_wireless-tag_sc01_plus_back_connection](images/esp32-s3_wireless-tag_sc01_plus_back_connection.png)

## Table of Contents
- [Hardware Requirements](#hardware-requirements)
- [Features](#features)
- [Setup Steps](#setup-steps)
- [Configuration](#configuration)
- [Languages](#languages)
- [Frost Protection Page](#frost-protection-page)
- [GPIO Pinout](#gpio-pinout)
- [Dependencies](#dependencies)
- [Development](#development)
- [Changelog](#changelog)
- [References](#references)

## Hardware Requirements
- **Display Module**: [Wireless-Tag WT32-SC01 PLUS](https://robu.in/product/wireless-tag-wt32-sc01-plus-esp32s3-based-3-5-inch-iot-display/) (ZX3D50CE08S-v15-USRC)
  - 3.5 inch LCD, 480 x 320 (used in landscape)
  - ST7796 driver, 8-bit MCU8080 (i80) interface, RGB565 color
  - Single touch capacitive FT6336U (I2C)
  - MCU: ESP32-S3
  - WiFi + BLE
  - 16MB flash
  - 2MB PSRAM (quad)

## Features

### Core Functionality
- Real-time temperature monitoring and control (dial, +/- buttons and knob)
- HVAC modes: Off, Auto, Cool, Heat, Dry, Fan Only
- Fan speed control with five configurable fan modes
- Temperature range and step match your climate entity
- Touch-sensitive interface

### Display Features
- Auto-dimming display, wakes on touch or when the AC changes
- Clock in 12h or 24h format
- Home Assistant connection indicator
- UI in several languages ([Languages](#languages))
- Optional frost protection page ([Frost Protection Page](#frost-protection-page))

## Setup Steps

### Prerequisites
1. Home Assistant with the ESPHome Device Builder app
2. WT32-SC01 Plus display module
3. USB-C cable for the first flash

### Install
1. Copy these files and folders to your ESPHome config folder (`/config/esphome/`):
   ```
   master-ac-display.yaml
   lang/
   fonts/
   ```
   For example:
   ```bash
   git clone https://github.com/GargIT/climate-control-display.git
   ```

2. Add these to `secrets.yaml`:
   ```yaml
   wifi_ssid: "Your_WiFi_SSID"
   wifi_password: "Your_WiFi_Password"
   fallback_password: "AP_Fallback_Password"
   ota_password: "OTA_Update_Password"
   api_key_b64: "Your_Base64_API_Key"
   ```

3. Edit the `CONFIG` block at the top of `master-ac-display.yaml` (see [Configuration](#configuration)).

4. Flash over USB the first time, then over WiFi.

## Configuration

All settings you normally change are in the `CONFIG` block of `substitutions`:

```yaml
substitutions:
  device_name: 'master-ac-display'
  friendly_name: 'Master AC Display'
  language: en                      # UI language: en, sv
  clock_24h: 'true'                 # 'false' gives 11:59:59PM

  climate_entity: climate.master_ac
  temp_min: '16'                    # min_temp of the climate entity
  temp_max: '30'                    # max_temp of the climate entity
  temp_step: '0.5'                  # target_temp_step of the climate entity
  power_on_mode: cool               # mode used when the power button turns the AC on
  hvac_auto_mode: heat_cool         # what the AUTO button sends: heat_cool or auto
  fan_mode_1: auto                  # fan_modes of the climate entity, in order
  fan_mode_2: low
  fan_mode_3: medium
  fan_mode_4: high
  fan_mode_5: strong
```

Take `temp_min`, `temp_max`, `temp_step`, the HVAC mode names and the fan modes from your climate entity's attributes (**Developer Tools → States** in Home Assistant). If they don't match, buttons will send values your AC rejects.

### Home Assistant Settings

After flashing, Home Assistant discovers the device automatically.

Make sure you have enabled **Allow the device to perform Home Assistant actions** in the device's ESPHome options. Without it the display shows the AC state but its buttons do nothing.

![ESPHome Device Configure](images/device-configure.png)

![Allow HA Actions](images/ha-action-submit.png)

#### Device Controls

![Device Controls](images/device-info.png)

### Display Settings
| Setting | Range | Default | Description |
|---------|--------|---------|-------------|
| Backlight Brightness | 5-100% | 80% | Screen brightness |
| Screen Timeout | 10-300s | 300s | Time before the screen dims and turns off |

### Fan Modes
| Slot | Icon | Default |
|------|------|---------|
| `fan_mode_1` | fan auto | auto |
| `fan_mode_2` | fan speed 1 | low |
| `fan_mode_3` | fan speed 2 | medium |
| `fan_mode_4` | fan speed 3 | high |
| `fan_mode_5` | fan | strong |

## Languages

Set `language:` in the `CONFIG` block. Included:

| Code | Language |
|------|----------|
| `en` | English |
| `sv` | Svenska |

All texts are in `lang/<code>.yaml`. To add a language, copy `lang/en.yaml` to `lang/<code>.yaml`, translate the values, and put every character outside A-Z, a-z and 0-9 that you use in `lang_glyphs` (for example `"ÄÖÜäöüß"`), so the fonts include it. Pull requests with new languages are welcome.

## Frost Protection Page

Many ACs have a frost protection (FP / 8 °C) mode that Home Assistant can't control. The display can show a frost protection page that drives a Home Assistant automation instead:

- A snowflake-thermometer button left of the WiFi icon opens the page. It is blue while frost protection is on.
- The page has an on/off button, the start and stop temperatures with +/- buttons, the room temperature and what the AC is doing.
- Below the start temperature the automation heats at the AC's lowest setpoint; above the stop temperature it turns the AC off.

The page is optional. Its button only appears when the frost protection switch exists in Home Assistant.

### Setup
1. Install [`home-assistant/frost_protection.yaml`](home-assistant/frost_protection.yaml) as a Home Assistant package (instructions in the file), or create the same helpers and automation in the UI.
2. If you use other entity IDs, set them in the `CONFIG` block:
   ```yaml
   fp_switch: input_boolean.frost_protection
   fp_start: input_number.frost_protection_start
   fp_stop: input_number.frost_protection_stop
   ```

## GPIO Pinout

### Display Interface
```
Display Control:
- SDA: GPIO6
- SCL: GPIO5
- DC:  GPIO0
- WR:  GPIO47
- RST: GPIO4
- BL:  GPIO45

Data Bus:
- D0: GPIO9
- D1: GPIO46
- D2: GPIO3
- D3: GPIO8
- D4: GPIO18
- D5: GPIO17
- D6: GPIO16
- D7: GPIO15
```

## Dependencies

### Required Software
- ESPHome 2026.11 or later (built-in `mipi_spi` display driver and LVGL 9; no external components needed)
- Home Assistant Core

### Optional Tools
- Visual Studio Code with the ESPHome extension

## Development

### Build and Upload
```bash
esphome run master-ac-display.yaml
```

### Building
```bash
esphome compile master-ac-display.yaml
```

### Uploading
```bash
esphome upload master-ac-display.yaml
```

### Debugging
```bash
esphome logs master-ac-display.yaml
```

### Clean
```bash
esphome clean master-ac-display.yaml
```

## Changelog

### v2.0.0 (2026-11)
- Updated for ESPHome 2026.11: built-in `mipi_spi` driver replaces the bundled external components, `esp32.cpu_frequency` and `esp32.flash_mode` replace the old sdkconfig and PlatformIO options.
- Fixed "Circular dependency detected" at compile time: the meter's `tick_style` indicator is not registered in current ESPHome; the set temperature is now shown as an `arc` indicator.
- Landscape orientation via LVGL `rotation: 90`; touch follows the rotation.
- Boot screen hides when Home Assistant connects; touching the screen wakes it and cancels dimming.
- Temperature knob sends the value when released instead of on every step.
- Temperature range, step, AUTO mode name, power-on mode and fan modes are configurable.
- Multi-language UI (`en`, `sv`) and 12h/24h clock.
- Optional frost protection page with a Home Assistant package.

The OTA update still uses `ota_password` so devices running v1 accept the update. ESPHome recommends OTA `encryption:` instead; switch after updating if you want, see the [ESPHome OTA docs](https://esphome.io/components/ota/esphome/#configuration-variables).

## References

### Documentation
- [WT32-SC01 Plus Datasheet](https://www.marutsu.co.jp/contents/shop/marutsu/datasheet/khe_WT32-SC01-PLUS.pdf)
- [ESPHome LVGL](https://esphome.io/components/lvgl/)
- [ESPHome FT63x6 Documentation](https://esphome.io/components/touchscreen/ft63x6.html)
- [LVGL Cookbook](https://esphome.io/cookbook/lvgl.html)

## License

MIT License

## Contributing

1. Fork the repository
2. Create your feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

---

Made with ❤️ by kpr. ESPHome 2026 update, languages and frost protection by GargIT.
