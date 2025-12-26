# Copilot Instructions for Personal Status Display (ESPHome)

This repo contains a single ESPHome firmware configuration for a Cheap Yellow Display (ESP32-2432S028) showing Mitch's current status and location. Most logic lives in YAML and LVGL widgets, split into small include files.

## Architecture
- **Main entry:** [main.yaml](main.yaml) includes all other files and defines boot behavior, Wi‑Fi/OTA/API, and `on_boot` flow.
- **Includes:**
  - [includes/constants.yaml](includes/constants.yaml): `$substitutions` for dimensions, timeouts, colors, and Unicode icons.
  - [includes/hardware.yaml](includes/hardware.yaml): ESP32 board, SPI buses, touchscreen, backlight `light`, screensaver `number` animation, and touch-to-resume logic.
  - [includes/fonts.yaml](includes/fonts.yaml): Google font and Material Symbols glyph set; icons must be listed here to render.
  - [includes/ui.yaml](includes/ui.yaml): LVGL config, pages (`page_main`, `page_screensaver`), labels `status_text` and `status_icon`, styles, idle handlers.
  - [includes/logic.yaml](includes/logic.yaml): Home Assistant `text_sensor`s for `input_select.mitch_status` and `person.mitch_talmadge`. `script.update_display` drives label text, icon, color, and forces redraw.
- **Data flow:** HA entities → `text_sensor.on_value` → `script.update_display` → LVGL label updates → `lvgl.widget.redraw`.
- **Idle/screen behavior:** `lvgl.on_idle` timers control backlight off, screensaver page, and LVGL pause; touchscreen `on_touch` resumes and returns to `page_main`.

## Developer Workflows
- **Always compile after changes:** After modifying any config, run the compile step (`ESPHome: Compile` task or `esphome compile main.yaml`) to catch build issues before handing work back.
- **Validate config:** `ESPHome: Validate Config` task (runs `esphome config main.yaml`).
- **Compile:** `ESPHome: Compile` task for the fixed main.yaml.
- **OTA upload:** `ESPHome: Upload (OTA)` task for the fixed main.yaml.
- **OTA mDNS tip:** If `mitch-personal-status-display.local` doesn’t resolve, upload with the device IP (e.g., `esphome upload main.yaml --device 10.10.10.180`).
- **Dashboard (optional):** `ESPHome: Dashboard` task starts `esphome dashboard /config` for local UI.
- **CLI equivalents:** From the repo root:
  - `esphome config main.yaml`
  - `esphome compile main.yaml`
  - `esphome upload main.yaml`
- **Logs:** Use ESPHome logs (serial/Wi‑Fi). LVGL `log_level: VERBOSE` in [includes/ui.yaml](includes/ui.yaml) can be noisy; lower to `INFO` if needed.

## Conventions & Patterns
- **Separation of concerns:** Keep hardware, UI, fonts, constants, and logic in their respective include files; reference via `<<: !include` in the main file.
- **Substitutions:** Use `$name` for shared values (e.g., `$idle_timeout`) and `${name}` for strings/colors in LVGL updates.
- **Icons & fonts:** If you add a new icon, include its Unicode in [includes/constants.yaml](includes/constants.yaml) and add it to the `glyphs` list in [includes/fonts.yaml](includes/fonts.yaml); otherwise it won’t render.
- **LVGL updates:** Always update both `status_text` and `status_icon` (text, `text_font`, `text_color`) and call `lvgl.widget.redraw` after scripted changes.
- **On-boot UX:** The firmware turns on the backlight, `lvgl.resume`, and shows `page_main` if idle/screensaver/backlight is off.
- **Screensaver color:** The rainbow background comes from `number_display_burnin_color` in [includes/hardware.yaml](includes/hardware.yaml) and applies only when `page_screensaver` is showing.

## Integrations & Secrets
- **Home Assistant:** Expects `input_select.mitch_status` and `person.mitch_talmadge`. Changes trigger `update_display`.
- **Secrets:** All credentials are referenced via `!secret` in [main.yaml](main.yaml). Store real values in [secrets.yaml](secrets.yaml). Avoid committing sensitive values to public repos.
- **Dependencies:** ESPHome CLI, LVGL (via ESPHome), Material Symbols Outlined font, Itim font (gfonts).

## Making Changes (Examples)
- **Add a new status (e.g., “Exercising”):**
  1. Add `${color_exercising}` and `${icon_exercising}` in [includes/constants.yaml](includes/constants.yaml).
  2. Add the icon value to `glyphs` in [includes/fonts.yaml](includes/fonts.yaml).
  3. Add a new `if` branch in `script.update_display` in [includes/logic.yaml](includes/logic.yaml) to set `status_text`, `status_icon`, `text_font`, and colors.
  4. Optionally set default UI text/icon in [includes/ui.yaml](includes/ui.yaml).
- **Adjust idle behavior:** Modify `$idle_timeout`, `$idle_burnin_start`, `$idle_burnin_stop` in [includes/constants.yaml](includes/constants.yaml); verify `on_idle` in [includes/ui.yaml](includes/ui.yaml).
- **Hardware tweaks:** Update pins, rotation, or display model in [includes/hardware.yaml](includes/hardware.yaml); keep `update_interval: never` and rely on redraws.

## Debugging Tips
- **Touch resume not working?** Check `touchscreen` calibration and `transform` in [includes/hardware.yaml](includes/hardware.yaml) and verify `on_touch` conditions.
- **Icons show as squares?** Ensure the icon codepoint is in `glyphs` of [includes/fonts.yaml](includes/fonts.yaml) and referenced via `${icon_*}`.
- **No color change in screensaver?** Confirm `page_screensaver` is showing and the `number_display_burnin_color` `on_value` applies `disp_bg_color`.