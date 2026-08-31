# PySide6 WebBrowser with H.264/MP4 Codec Support

A lightweight open-source desktop application built with Python and PySide6 (Qt for Python). This project features an enhanced configuration of **Qt WebEngine** that enables full playback support for proprietary video and audio codecs such as **H.264, MP4, and AAC** (e.g., for watching YouTube, Netflix, or HTML5 video streams), which are normally disabled in the stock PySide6 distribution.

## Features

- **PySide6 & Qt WebEngine v6.11.x** core engine (latest available 6.11 patch release — see [Build Notes](#build-notes--known-issues) below for why this isn't hard-pinned to a specific patch).
- **Proprietary Codecs Enabled:** Native playback support for H.264, MP4, and AAC.
- **Automated Build Pipeline:** Binaries are compiled safely via GitHub Actions using the LGPL-compliant internal Chromium multimedia stack. Currently **Windows-only** (`windows-latest` runner); Linux/macOS jobs are not yet part of the workflow.

---

## Installation & Setup

Because proprietary codecs are bound by regional patent restrictions, the pre-built binaries cannot be bundled directly inside the main application code. Instead, users must download the compiled plugin binary separately.

### Step 1: Install the Main Application
Clone this repository and install the standard dependencies:
```bash
git clone https://github.com/danxdigitalsolution-stack/web-engine
cd YOUR_REPO_NAME


### Step 2: Download the Custom WebEngine Binary
1. Go to the **Releases** tab of this GitHub repository.
2. Download the compressed binary archive for your Operating System — currently `qtwebengine-windows-6.11.x-codecs.zip` (Windows only; see [Build Notes](#build-notes--known-issues)).
3. Extract the contents of the archive.

### Step 3: Replace the Stock PySide6 Binary (Opt-In)
To activate the codecs, copy the extracted custom binaries and overwrite the default files in your local Python environment:

#### 🐧 On Linux:
- Move `libQt6WebEngineCore.so.6` and `libQt6WebEngineWidgets.so.6` into your virtual environment at:  
  `./.venv/lib/python3.x/site-packages/PySide6/Qt/lib/`
- Move the contents of the `libexec/` folder into:  
  `./.venv/lib/python3.x/site-packages/PySide6/Qt/libexec/`

#### 🪟 On Windows (If using Windows build artifacts):
- Move `Qt6WebEngineCore.dll` and `Qt6WebEngineWidgets.dll` into your virtual environment at:  
  `.\.venv\Lib\site-packages\PySide6\`
- Move `QtWebEngineProcess.exe` into:  
  `.\.venv\Lib\site-packages\PySide6\`

---

## Build Notes & Known Issues

- **Why the Qt version isn't hard-pinned:** `download.qt.io` changed its Windows desktop directory layout starting with Qt 6.11.0 (per-toolchain folders instead of the old nested structure). The official `aqtinstall` PyPI release (3.3.0) doesn't understand the new layout yet, so a hard-pinned version like `6.11.2` fails to resolve. The fix is merged upstream ([aqtinstall#1000](https://github.com/miurahr/aqtinstall/pull/1000)) but not yet released to PyPI, so `build.yml` installs `aqtinstall` directly from the `master` branch on GitHub and requests the bare `6.11` series, letting `aqt` resolve to whatever the newest published 6.11.x patch is at build time.
- **Dynamic Qt path detection:** because the installed patch version floats, the workflow detects the actual installed folder under `Qt\` after `aqt install-qt` runs (instead of hardcoding a path like `Qt\6.11.2\msvc2022_64`) and exports it as `QT_DIR` / `QT_VERSION` env vars for later steps. An earlier revision hardcoded the path, which broke `find_package(Qt6)` in CMake ("Could not find a package configuration file... Qt6Config.cmake") whenever aqt resolved to a patch other than 6.11.2.
- **`CMAKE_PREFIX_PATH` set explicitly:** just adding Qt's `bin/` to `PATH` isn't enough for CMake's Config-mode `find_package(Qt6 ...)` to locate `Qt6Config.cmake`. The configure step now passes `-DCMAKE_PREFIX_PATH="%QT_DIR%"` explicitly.
- **Qt WebEngine source tag matches Qt Base exactly:** the WebEngine source clone now checks out `v<QT_VERSION>` (the exact patch that `aqt` actually installed) instead of a hardcoded tag, avoiding qtbase/qtwebengine version mismatches.
- **Source clone is cached:** cloning `qtwebengine --recursive` pulls in the full Chromium submodule tree (multiple GB) and is by far the slowest part of the pipeline. The workflow now caches the `qtwebengine/` directory via `actions/cache`, keyed on the resolved `QT_VERSION`, so re-runs against the same patch skip the clone entirely. The cache key changes automatically whenever the resolved Qt patch changes.
- **Track record:** Once `aqtinstall` ships a stable release containing the folder-layout fix, switch the workflow back to `pip install -U aqtinstall` from PyPI and optionally re-pin to an exact patch version if reproducibility matters more than always getting the latest codecs/security fixes.
- **`-m qtwebengine` intentionally omitted:** the workflow only installs Qt Base via `aqt`; Qt WebEngine itself is compiled from source in the following step, so the prebuilt WebEngine module isn't needed.

---

## Verification

To verify that the custom codecs are working correctly, run the application and navigate to an HTML5 capability test site like [https://html5test.co.za](https://html5test.co.za). The status for **H.264 Support** and **AAC Support** should display a green checkmark.

---

## License & Disclaimers

### License
The source code written for this application is open-source and distributed under the **MIT License**. You are free to modify and redistribute the code.

### PySide6 and Qt Compliance
This application dynamically links to PySide6, which is licensed under the **GNU LGPLv3**. No modifications have been made to the core PySide6 Python wrapper source files.

### Third-Party Patent & Codec Disclaimer
This repository provides automated build scripts (GitHub Actions) to compile the open-source Qt WebEngine (Chromium) source code. 
- The activation of proprietary media codecs (H.264/MP4) is handled via a **user-initiated, manual opt-in mechanism**.
- This project does not directly distribute, bundle, or monetize copyrighted binary decoders. 
- End-users are solely responsible for ensuring compliance with local commercial patent laws (such as those managed by the MPEG-LA consortium) if this software is used outside of private, non-commercial, or educational environments.
