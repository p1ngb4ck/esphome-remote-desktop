# `pr/remote-desktop-close-action`

One part of the topic repo p1ngb4ck/esphome-remote-desktop. The whole topic, with its description, is the branch `main`.

Change of this branch: the close gesture a configured key binding (`close_action`, default `home`; Back stays a normal key)

Based on these branches, whose changes it holds and builds on:

- `pr/display-video-output-page-draw` of p1ngb4ck/esphome-esp-video: display: page draw calls (`page_rect`, `page_fill`, `page_pixels`, `page_copy`, `page_read`, `page_show`, `page_bpp`, `canvas_point`)
- `pr/storage` of p1ngb4ck/esphome-storage-interface: the `storage` component: registry, path / filesystem / network / raw storages, the asynchronous worker (jobs, streams), `storage.*` actions, PSRAM transfer buffer, change feed, ISO 9660 mount
- `pr/usb-host-refactor` of p1ngb4ck/esphome-usb-host: routing by device class to the drivers (named > vendor table > class, interfaces per driver), several devices per client, one shared USB task, unified transfer engine (bulk, interrupt, control, isochronous streams), driver `blacklist`, host channel budget per controller, the patched espressif/usb component (per-port FIFO bias, `fsls_only`, external hub port requests, root port reset recovery)
