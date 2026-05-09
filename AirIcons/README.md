# AirIcons

The ESPHome YAML references local PNG assets for the boot logo, sensor icons, settings icons, battery icons, and the particulate-matter source graphic.

Expected files:

- `AirMikologo.png` - boot splash/logo
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

These images are intentionally not included. Add your own PNGs with these names, or remove/comment the matching `image:` entries and display calls in `airmikol.yaml`.

The splash screen uses `AirMikologo.png`; replace it with your own logo or remove the `it.image(...)` line on the boot page.

