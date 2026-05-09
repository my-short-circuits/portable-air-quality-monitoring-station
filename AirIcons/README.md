# AirIcons

The ESPHome YAML references local PNG assets for the boot logo, sensor icons, settings icons, battery icons, and the particulate-matter source graphic.

Expected files:

- `logo.png` - boot splash/logo, intentionally not included
- `CO2icon.png`
- `COicon.png`
- `NH3icon.png`
- `NO2icon.png`
- `TVOCicon.png`
- `PMicon.png`
- `TEMPicon.png`
- `HUMIicon.png`
- `SETTINGSicon.png`
- `BATTERYicon.png`
- `MINIBATTERYicon.png`
- `CLEANicon.png`
- `WIFIicon.png`
- `SOUNDicon.png`
- `SCREENicon.png`
- `LEDicon.png`
- `INFOicon.png`
- `PMsources.png`
- `TIMEOUTicon.png`
- `PLUGicon.png`

The shared UI icons are included. The logo is intentionally not included; add your own `logo.png`, or remove/comment the matching `image:` entry and boot-page display call in `portable-air-quality-monitoring-station.yaml`.

The splash screen uses `logo.png`; replace it with your own logo or remove the `it.image(...)` line on the boot page.
