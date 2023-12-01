# eclinic-notifications

Desktop Application written in [Rust](https://www.rust-lang.org/) and [Tauri](https://tauri.app/) for handling server-sent event
notifications from a EMR application. The frontend uses sveltekit and TS.

# Development

`npm run tauri dev`

# Build / release

`npm run tauri build`

Close application from notification tray.

# Windows

Requires **Microsoft Visual Studio C++ Build Tools**

The easiest way is to install [Build Tools for Visual Studio 2022](https://visualstudio.microsoft.com/visual-cpp-build-tools/) . When asked which workloads to install, ensure **"C++ build tools"** and the **Windows 10 SDK** are selected.

# Linux

See [Prerequisites](https://tauri.app/v1/guides/getting-started/prerequisites) on the tauri website.

```bash

sudo apt update
sudo apt install libwebkit2gtk-4.0-dev \
    build-essential \
    curl \
    wget \
    file \
    libssl-dev \
    libgtk-3-dev \
    libayatana-appindicator3-dev \
    librsvg2-dev
```
