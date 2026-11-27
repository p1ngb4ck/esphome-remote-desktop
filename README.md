# esphome-remote-desktop

## Purpose
`remote_desktop`: a VNC and SPICE client app for the ESP32-P4. The remote screen is drawn into an LVGL canvas;
keyboards, mice and touch control the remote machine; SPICE sound plays on a speaker; USB devices can be redirected
to the remote machine. No server in between - the device connects directly.

## How it works
The remote screen goes through the display's video output into an LVGL canvas with an external buffer (Tight JPEG
and SPICE MJPEG through the hardware JPEG decoder). HID inputs (`hid_ids`) and the touchscreen are sent as keys and
pointer events. Connections are started with actions (`remote_desktop.connect`, saved connections); a select platform
redirects USB devices (usbredir with SPICE, USB/IP with VNC). Display / LVGL come from esphome-mipi-display, `hid`
from esphome-hid.

## Quick start
Chips: ESP32-P4

Needs: [esphome-mipi-display](https://github.com/p1ngb4ck/esphome-mipi-display),
[esphome-hid](https://github.com/p1ngb4ck/esphome-hid), [esphome-network](https://github.com/p1ngb4ck/esphome-network),
[esphome-esp32-platform](https://github.com/p1ngb4ck/esphome-esp32-platform),
[esphome-storage-drivers](https://github.com/p1ngb4ck/esphome-storage-drivers)

```yaml
external_components:
  - source: github://p1ngb4ck/esphome-remote-desktop@main
    components: [remote_desktop]
  - {source: github://p1ngb4ck/esphome-mipi-display@main, components: [display, lvgl, mipi, mipi_dsi]}
  - {source: github://p1ngb4ck/esphome-hid@main, components: [hid]}
  - {source: github://p1ngb4ck/esphome-network@main, components: [network, ethernet]}
  - {source: github://p1ngb4ck/esphome-esp32-platform@main, components: [esp32]}
  - {source: github://p1ngb4ck/esphome-storage-drivers@main, components: [esp_ldo]}
esp_ldo: [{channel: 3, voltage: 2.5V}]
display: [{platform: mipi_dsi, id: main_display, model: JC1060P470, enable_pin: GPIO22}]
ethernet: {type: W5500, clk_pin: GPIO18, mosi_pin: GPIO21, miso_pin: GPIO14, cs_pin: GPIO13, interrupt_pin: GPIO4}
hid:
lvgl:
  id: lvgl_main
  displays: [main_display]
  video_output: {canvases: [rd_canvas]}
  widgets:
    - canvas: {id: rd_canvas, width: 1024, height: 600, external_buffer: true}
remote_desktop: {lvgl_id: lvgl_main}
```

Use your board's pins and display model. Connect: `remote_desktop.connect: {host: 192.168.1.50, port: 5900}`;
`hid_ids` from e.g. `usb_hid` (esphome-usb-drivers); `connections_file` needs esphome-storage-interface.

## Main options
| Option | What it does | Default |
|---|---|---|
| [`lvgl_id`](CONFIGURATION.md#remote_desktop) | LVGL instance shown through the video output | required |
| [`hid_ids`](CONFIGURATION.md#remote_desktop) | up to 4 HID inputs (keyboards, mice) sent to the remote | `[]` |
| [`touchscreen_id`](CONFIGURATION.md#remote_desktop) | touch as pointer; two-finger hold ends it | - |
| [`speaker`](CONFIGURATION.md#remote_desktop) | SPICE sound output | - |
| [`connections`](CONFIGURATION.md#remote_desktop) | saved connections, 1-16 | `8` |
| [`connections_file`](CONFIGURATION.md#remote_desktop) | a storage file instead of the preferences | - |
| [`close_action`](CONFIGURATION.md#remote_desktop) | key binding action that ends the session when held | `home` |
| [`remote_desktop.connect`](CONFIGURATION.md#actions-and-condition) | `host`, `port`, `protocol` | `vnc` |
| [`select` platform](CONFIGURATION.md#usb-redirection-select) | up to 4 USB redirection slots (needs `usb_host`) | - |

## Catalog
- Components:
  - `remote_desktop`:
    [folder](https://github.com/p1ngb4ck/esphome-remote-desktop/tree/main/esphome/components/remote_desktop),
    [configuration](CONFIGURATION.md#remote_desktop), [actions](CONFIGURATION.md#actions-and-condition),
    [USB redirection select](CONFIGURATION.md#usb-redirection-select)
- Branches:
  - [`main`](https://github.com/p1ngb4ck/esphome-remote-desktop/tree/main): the whole topic
  - [`pr/remote-desktop`](https://github.com/p1ngb4ck/esphome-remote-desktop/tree/pr/remote-desktop)
  - [`pr/remote-desktop-close-action`][close-action]
- Needed repos: [esphome-mipi-display](https://github.com/p1ngb4ck/esphome-mipi-display) (display, lvgl, mipi_dsi),
  [esphome-hid](https://github.com/p1ngb4ck/esphome-hid) (hid),
  [esphome-network](https://github.com/p1ngb4ck/esphome-network) (network, ethernet / wifi),
  [esphome-esp32-platform](https://github.com/p1ngb4ck/esphome-esp32-platform) (esp32),
  [esphome-storage-drivers](https://github.com/p1ngb4ck/esphome-storage-drivers) (esp_ldo)
- Optional: [esphome-storage-interface](https://github.com/p1ngb4ck/esphome-storage-interface) (`connections_file`),
  [esphome-usb-host](https://github.com/p1ngb4ck/esphome-usb-host) (USB redirection select),
  [esphome-usb-drivers](https://github.com/p1ngb4ck/esphome-usb-drivers) (`usb_hid` as HID source)
- All topics together: `github://p1ngb4ck/esphome@dev`

[close-action]: https://github.com/p1ngb4ck/esphome-remote-desktop/tree/pr/remote-desktop-close-action

---

# Detailed version

`remote_desktop`: a VNC and SPICE client app for the ESP32-P4. The remote screen is drawn into an LVGL canvas
through the display's video output (Tight JPEG and SPICE MJPEG through the hardware JPEG decoder); keyboards, mice
and the touchscreen control the remote machine; SPICE sound plays on an ESPHome speaker; USB devices can be
redirected to the remote machine. No server in between - the device connects directly.

Needs: display and lvgl from [esphome-mipi-display](https://github.com/p1ngb4ck/esphome-mipi-display), `hid` (and
the LVGL HID input) from [esphome-hid](https://github.com/p1ngb4ck/esphome-hid), `esp32` from
[esphome-esp32-platform](https://github.com/p1ngb4ck/esphome-esp32-platform); `storage` from
[esphome-storage-interface](https://github.com/p1ngb4ck/esphome-storage-interface) only for `connections_file`;
`usb_host` from [esphome-usb-host](https://github.com/p1ngb4ck/esphome-usb-host) only for the USB redirection select.
A HID source for `hid_ids`: e.g. `usb_hid` of [esphome-usb-drivers](https://github.com/p1ngb4ck/esphome-usb-drivers).
Take each component from its owner repo, not from copies in this repository.

## Examples

```yaml
external_components:
  - source: github://p1ngb4ck/esphome-remote-desktop@main
    components: [remote_desktop]
  - source: github://p1ngb4ck/esphome-mipi-display@main
    components: [display, lvgl]
  - source: github://p1ngb4ck/esphome-hid@main
    components: [hid]
  - source: github://p1ngb4ck/esphome-storage-interface@main
    components: [storage]
  - source: github://p1ngb4ck/esphome-usb-host@main
    components: [usb_host]
  - source: github://p1ngb4ck/esphome-esp32-platform@main
    components: [esp32]

remote_desktop:
  id: rdp
  lvgl_id: lvgl_main
  hid_ids: [keyboard, mouse]
  touchscreen_id: touch
  speaker: main_speaker

button:
  - platform: template
    name: Connect to server
    on_press:
      - remote_desktop.connect:
          host: 192.168.1.50
          port: 5900
          password: !secret vnc_password
```

## More

**[CONFIGURATION.md](CONFIGURATION.md)**: every option and action - protocols (`vnc`, `spice`, `auto`), saved
connections, the close gesture, the connection select.

## Branches

One branch per change; `main` contains every branch. Base: ESPHome `dev` at `a5a75738ad`.

| Branch | Change | Needs | Chips |
|---|---|---|---|
| `pr/remote-desktop` | `remote_desktop` (see below) | see below | P4 |
| `pr/remote-desktop-close-action` | the close gesture a configured key binding (see below) | `pr/remote-desktop` | P4 |

`pr/remote-desktop`: VNC (RFB, Tight JPEG through the hardware decoder) and SPICE (MJPEG streams, JPEG images through
the hardware decoder) on a page canvas of the video output; keys / mouse / touches, cursor, SPICE sound, USB
redirection (usbredir; USB/IP with VNC), saved connections, probing (`protocol: auto`), close gestures. It needs
`pr/display-video-output-page-draw` ([esphome-esp-video](https://github.com/p1ngb4ck/esphome-esp-video)),
`pr/lvgl-hid-input` ([esphome-hid](https://github.com/p1ngb4ck/esphome-hid)), `pr/storage`
([esphome-storage-interface](https://github.com/p1ngb4ck/esphome-storage-interface)) and `pr/usb-host-refactor`
([esphome-usb-host](https://github.com/p1ngb4ck/esphome-usb-host)).

`pr/remote-desktop-close-action`: the close gesture is a configured key binding (`close_action`, default `home`;
Back stays a normal key).

All topics together: `github://p1ngb4ck/esphome@dev`. `external_components` replaces components only; the ESPHome
core of the build host stays as installed.

## License

- ESPHome's own files, also where this repository changes them: the [ESPHome License](LICENSE) (MIT for the Python
  code and everything else, GPLv3 for the C++ runtime files).
- New files written for this repository (new components, their code, tests and documentation): MIT,
  [LICENSE-MIT](LICENSE-MIT).

## Thanks

Everyone this project builds on: [ATTRIBUTIONS.md](https://github.com/p1ngb4ck/esphome/blob/dev/ATTRIBUTIONS.md).

## Branch structure

Each `pr/` branch holds one part of this topic on top of upstream and says in its `BRANCH.md` what it is based on.
`main` holds all of them. `pr/remote-desktop` is based on `pr/display-video-output-page-draw` of
p1ngb4ck/esphome-esp-video, `pr/storage` of p1ngb4ck/esphome-storage-interface and `pr/usb-host-refactor` of
p1ngb4ck/esphome-usb-host; it also needs `pr/lvgl-hid-input` of p1ngb4ck/esphome-hid (see "Branches"), which its
`BRANCH.md` does not name as a base.

| Branch | Based on |
|---|---|
| `pr/remote-desktop` | the three branches named above |
| `pr/remote-desktop-close-action` | `pr/remote-desktop` |
