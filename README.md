# Portable Air Quality Monitoring Station

Portable Air Quality Monitoring Station is an ESPHome indoor air quality monitor built around a FireBeetle 2 ESP32-E, a color touchscreen, and a dense set of air-quality sensors. It combines true NDIR CO2 measurement, particulate monitoring, VOC/NOx indexing, temperature and humidity correction, relative gas trend sensing, battery tracking, local alerts, and Home Assistant entities in one device.

The project is designed for a compact custom build: the components are connected through a purpose-built PCB and arranged to fit inside a 3D printed enclosure. The YAML also includes a full on-device touch UI, so the monitor remains useful even when you are not looking at Home Assistant.

## What It Does

- Shows a local 320x240 touchscreen dashboard for air quality, particulate matter, climate, gas trends, battery, and system status.
- Exposes ESPHome/Home Assistant entities for CO2, PM1.0/2.5/4/10, particle counts, VOC index, NOx index, temperature, humidity, dew point, PM2.5 AQI, an overall IAQ score, battery state, and controls.
- Uses a status NeoPixel to show air quality at a glance.
- Uses a buzzer for local alerts, with sound control from the screen or Home Assistant.
- Tracks daily min/max values and 4-hour on-device graphs.
- Supports CO2 forced calibration, SPS30 fan auto-clean, gas baseline setting, screen timeout, display brightness, LED brightness, battery save mode, dark mode, Wi-Fi toggle, and optional deep sleep.
- Estimates battery percentage and rough remaining runtime from a LiPo voltage divider.

This is not a certified safety instrument. Do not use it as a life-safety CO alarm, smoke detector, or regulatory air monitor.

## Sensor Stack

| Sensor | Role | Notes |
| --- | --- | --- |
| Sensirion SCD41 | CO2, internal temp/humidity compensation | True NDIR optical CO2 sensor. The YAML uses low-power periodic measurement, automatic self-calibration, daily min/max tracking, and manual fresh-air calibration. |
| Sensirion SPS30 | Particulate matter | Laser particle sensor with PM1.0, PM2.5, PM4.0, PM10, particle counts, typical particle size, warm-up handling, and manual fan auto-clean. |
| Sensirion SGP4x | VOC and NOx index | Reports VOC and NOx indexes. The YAML compensates it using SCD41 temperature/humidity. |
| Sensirion SHT3x | Primary temperature/humidity | Used for the main indoor climate reading. The YAML applies manual offset plus dynamic self-heating correction. |
| CCS811 | Secondary TVOC/eCO2 | Used as an additional VOC-style reading. eCO2 is not the same as true CO2; the SCD41 is the primary CO2 source. |
| ADS1115 | Analog gas ADC | Reads relative CO, NH3, and NO2 channels. These are baseline-relative trends, not exact ppm readings. |
| ESP32 internal temp | Self-heating correction | Helps compensate the temperature reading when the ESP32 warms the enclosure. |
| Battery ADC | LiPo voltage estimate | Reads a divided battery voltage on GPIO34 and maps it to a percentage curve. |

## Hardware

Core hardware used by the YAML:

- DFRobot FireBeetle 2 ESP32-E or compatible ESP32 board using the `firebeetle32` ESPHome board profile
- ILI9341 320x240 SPI TFT
- XPT2046 resistive touchscreen controller
- SCD41 CO2 sensor
- SPS30 particulate sensor
- SGP4x VOC/NOx sensor
- SHT3x temperature/humidity sensor at `0x44`
- CCS811 TVOC/eCO2 sensor at `0x5A`
- ADS1115 ADC at `0x48`
- Analog gas channels for CO, NH3, and NO2
- 1x WS2812/NeoPixel status LED
- PWM buzzer
- LiPo battery, default capacity set to `4000` mAh
- Battery voltage divider feeding ESP32 ADC
- Custom PCB to connect the components and package them into the printed enclosure

## Pinout

| Function | ESP32 pin | Notes |
| --- | --- | --- |
| I2C SDA | GPIO21 | Shared by SCD41, SHT3x, SGP4x, SPS30, CCS811, ADS1115 |
| I2C SCL | GPIO22 | Shared I2C clock |
| SPI CLK | GPIO18 | TFT and touch |
| SPI MOSI | GPIO23 | TFT and touch |
| SPI MISO | GPIO19 | Touch/TFT bus |
| TFT CS | GPIO14 | ILI9341 chip select |
| TFT DC | GPIO25 | ILI9341 data/command |
| TFT reset | GPIO1 | Shared with UART TX using `allow_other_uses`; adjust if your wiring differs |
| Touch CS | GPIO4 | XPT2046 chip select |
| Backlight PWM | GPIO12 | Drives TFT backlight, usually through a transistor/MOSFET if required |
| Status LED | GPIO5 | WS2812/NeoPixel data |
| Buzzer PWM | GPIO13 | PWM output for beep alerts |
| Battery ADC | GPIO34 | Expects a voltage divider; YAML multiplies ADC voltage by 2.0 |
| UART RX | GPIO3 | Reserved/configured at 9600 baud |
| UART TX | GPIO1 | Reserved/configured at 9600 baud and shared with TFT reset in this design |
| GPIO output | GPIO15 | Defined as `gpio_15`; not actively used elsewhere in the YAML |

ESPHome warns that GPIO15, GPIO12, and GPIO5 are ESP32 strapping pins. They can work, but avoid external pull-ups or pull-downs that would force the wrong boot state.

## I2C Devices

| Device | Address |
| --- | --- |
| ADS1115 | `0x48` |
| SHT3x | `0x44` |
| SPS30 | `0x69` |
| CCS811 | `0x5A` |
| SCD41 | ESPHome default |
| SGP4x | ESPHome default |

Run with `i2c.scan: true` during bring-up and verify that the expected addresses appear in the logs.

## ADS1115 Analog Channels

| ADS1115 channel | YAML entity | Intended signal |
| --- | --- | --- |
| A0 to GND | `NO2 relative` | Relative NO2 channel |
| A1 to GND | `NH3 relative` | Relative NH3 channel |
| A2 to GND | `CO relative` | Relative CO channel |

The gas page and `Gas Index` are trend-oriented. Let the unit warm up, expose it to normal clean room air, and set a baseline from the screen or Home Assistant. These gas readings are useful for relative changes, not calibrated concentration reporting.

## Files

- `portable-air-quality-monitoring-station.yaml` - sanitized ESPHome configuration
- `secrets.yaml.example` - template for private credentials
- `fonts/README.md` - font requirements
- `AirIcons/README.md` - image/logo/icon requirements

No secrets, private Wi-Fi information, photos, or binary image assets are included.

## Required Setup Before Flashing

1. Install ESPHome `2026.4` or newer.
2. Copy `secrets.yaml.example` to `secrets.yaml`.
3. Fill in your own Wi-Fi SSID, Wi-Fi password, ESPHome API encryption key, OTA password, and fallback AP password.
4. Edit the substitutions at the top of `portable-air-quality-monitoring-station.yaml`:
   - `device_name`
   - `friendly_name`
   - `co2_cal_ppm`
   - `battery_mah`
   - `timezone`
   - `outdoor_weather_entity`
5. Add your own font files:
   - `fonts/PNB.ttf`
   - `fonts/PNR.ttf`
6. Add your own PNG assets in `AirIcons/`, or remove/comment the `image:` entries and display calls that use them.
7. Review the pinout and change pins if your PCB or hand wiring differs.
8. Review the CCS811 `baseline:` value. Replace it with your own learned baseline or remove it.

## Images and Fonts

The YAML uses local PNG images and local `.ttf` fonts. They are intentionally ignored by git because they are personal or licensed assets.

The boot splash uses:

- `AirIcons/logo.png`

The particulate info page uses:

- `AirIcons/PMsources.png`

The YAML also defines multiple icon PNGs for future or alternate UI use. If any referenced image is missing, ESPHome can fail during compile. Replace the files with your own assets, or comment out the matching entries and any `it.image(...)` drawing calls.

The font filenames are `PNB.ttf` and `PNR.ttf`. If these are Proxima Nova or another commercial font, make sure you have the correct license. You can also switch the YAML to open fonts such as Inter, Roboto, or Noto Sans if you prefer.

## Home Assistant Notes

The ESPHome API exposes the monitor as a Home Assistant device with sensors, binary sensors, switches, numbers, buttons, lights, and diagnostics.

The YAML includes optional Home Assistant weather reads:

- `Outdoor Temp`
- `Outdoor Humidity`
- `Weather`

By default these use the `outdoor_weather_entity` substitution, set to `weather.home`. Change that to your own weather entity, or remove the outdoor weather sensors and related display references if you do not want the device to depend on Home Assistant for outdoor conditions.

## Calibration and Operation

### CO2

The SCD41 is the primary CO2 source. It uses automatic self-calibration and includes a manual `Calibrate CO2` button. For manual calibration, put the unit in fresh outdoor air, let it stabilize, and calibrate to the value in `co2_cal_ppm`. The default is `420`.

### Particulate Matter

The SPS30 reports PM mass concentrations, particle counts, and typical particle size. The UI shows warm-up status and exposes a `Clean PM Sensor` action that runs the SPS30 fan auto-clean routine.

### VOC and NOx

The SGP4x produces VOC and NOx indexes rather than direct ppb/ppm values. The IAQ score uses these indexes together with CO2, PM2.5, gas trend, and humidity.

### Temperature and Humidity

The SHT3x is treated as the main temperature/humidity source. The YAML applies:

- A persistent manual baseline offset, default `-1.4 C`
- A dynamic self-heating correction based on ESP32 die temperature
- Humidity recalculation after temperature correction

Tune `Temperature Baseline Offset` and `Self-Heat Strength` after comparing against a known-good reference.

### Gas Baseline

The CO, NH3, and NO2 channels are relative gas signals through the ADS1115. They require a clean-air baseline. Let the device warm up for about 20 minutes in normal clean air, then use `Set Gas Baseline`. The YAML can also auto-seed a baseline after warm-up if one has not been set.

## Battery Notes

Battery voltage is read on GPIO34 and multiplied by `2.0`, so the hardware is expected to divide the LiPo voltage roughly in half before it reaches the ESP32 ADC. Keep the ADC input within ESP32-safe voltage limits.

The battery percentage curve is an estimate, not coulomb counting. Runtime is estimated from `battery_mah` and assumed current draw:

- About `150 mA` with Wi-Fi enabled
- About `80 mA` with Wi-Fi disabled
- Reduced further in battery save mode

Use properly protected LiPo cells and safe charging hardware.

## Flashing

From this directory:

```bash
esphome run portable-air-quality-monitoring-station.yaml
```

For a first flash over USB, connect the FireBeetle 2 ESP32-E by USB and choose the serial target when ESPHome prompts. After adoption, OTA updates can use the configured `ota_password`.

## Troubleshooting

- Missing fonts or PNG files usually cause compile-time errors. Add the assets or remove the references.
- If outdoor weather shows `waiting`, update `outdoor_weather_entity` to a real Home Assistant weather entity.
- If touch coordinates are wrong, recalibrate the XPT2046 values under `touchscreen.calibration`.
- If battery percentage is wrong, verify the voltage divider ratio and adjust the `multiply: 2.0` filter.
- If gas readings always show `Needs baseline`, let the analog gas sensors warm up and run `Set Gas Baseline`.
- If the TFT reset pin conflicts with your board wiring, move `display.reset_pin` off GPIO1 and update the wiring table.

## Publishing Safely

Before publishing your fork or modified copy:

- Confirm `secrets.yaml` is not committed.
- Search for your Wi-Fi SSID, passwords, API keys, exact home location, and private Home Assistant entity names.
- Replace or remove personal logos and icons.
- Confirm that you have rights to any fonts or images you include.
- Add a license file if you want others to reuse the project under a specific open-source license.
