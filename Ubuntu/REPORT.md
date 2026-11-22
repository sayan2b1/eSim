# eSim – Ubuntu 25.04 Support Report

This report documents the issues faced while installing **eSim** and its dependency **nghdl** on Ubuntu 25.04, along with the fixes implemented.

---

## 🔧 nghdl

### 1. LLVM

* **Issue:**
  GHDL does not support the latest version of LLVM (`llvm-20.1`) available in Ubuntu 25.04.
* **Fix:**
  Modified the installer script to install `llvm-18.1`, which is supported.

---

### 2. InstallGHDL

* **Issue:**
  The installer script uses `/usr/bin/llvm-config` assuming `llvm-20.1`.
  Example command:

  ```bash
  ./configure --with-llvm-config=/usr/bin/llvm-config
  ```
* **Fix:**
  Replaced with the correct path for LLVM 18.1:

  ```bash
  ./configure --with-llvm-config=/usr/bin/llvm-config-18
  ```

---

### 3. Gtk Canberra

* **Issue:**
  `libcanberra-gtk-module` package is no longer available in Ubuntu 25.04.
* **Fix:**
  Modified installer to use only `libcanberra-gtk3-module`.

---

## 🔧 eSim

### 1. install-eSim.sh

* **Issue:**
  Ubuntu 25.04 was not supported in the main installer script.
* **Fix:**
  Added Ubuntu 25.04 support.

---

### 2. New Installer Script

* **Added:**
  `install-eSim-scripts/install-eSim-25.04.sh`

* **Differences from `install-eSim-24.04.sh`:**

  #### installKicad function:

  * Defines PPA for Ubuntu 25.04:

    ```bash
    ppa:kicad/kicad-8.0-releases
    ```

    (same as Ubuntu 24.04).
  * Installed specific version:

    ```bash
    kicad=8.0.8+dfsg-1
    ```

    instead of the latest version (latest causes dependency issues with `libgit2-1.8`).

---

---

# eSim 2.5 – Ubuntu 26.04 LTS Support Report

This section documents all dependency, compilation, and runtime issues encountered while installing **eSim 2.5** and **NGHDL** on **Ubuntu 26.04 LTS (`resolute`)**, along with the fixes implemented.

---

## 🔧 eSim (`install-eSim.sh` & `install-eSim-26.04.sh`)

### 1. Syntax Error & Stray Config Lines in `install-eSim.sh`
- **Issue:** Running `./install-eSim.sh --install` failed immediately with:
  `./install-eSim.sh: line 70: syntax error near unexpected token '}'`
  `run_version_script()` contained stray `config.ini` lines and lacked a closing `fi` for `if [[ -f "$SCRIPT" ]]; then`.
- **Fix:** Removed the misplaced lines from `run_version_script()` and properly closed the `if` block with `else` error handling and `fi`.

### 2. Ubuntu 26.04 Version Detection Failure (`install-eSim.sh`)
- **Issue:** `FULL_VERSION=$(lsb_release -d | grep -oP '\d+\.\d+\.\d+')` returned empty on `Ubuntu 26.04 LTS` (two-part version string), and `case $VERSION_ID` exited with `Unsupported Ubuntu version: 26.04 ()`.
- **Fix:** Updated the regex to `\d+\.\d+(\.\d+)?` and added a `"26.04"` case routing to `install-eSim-scripts/install-eSim-26.04.sh`.

### 3. KiCad PPA & Package Version Pinning (`installKicad`)
- **Issue:** Ubuntu 26.04 fell back to `ppa:kicad/kicad-6.0-releases` (unsupported on `resolute`) and pinned `kicad=8.0.8+dfsg-1`, which is unavailable in Ubuntu 26.04 repositories.
- **Fix:** Updated `installKicad()` in `install-eSim-26.04.sh` to install Ubuntu 26.04's native `kicad` (KiCad 9) packages directly from the official `universe` repository without adding legacy PPAs.

### 4. Hardcoded KiCad `6.0` Library Path (`copyKicadLibrary`)
- **Issue:** `copyKicadLibrary()` copied `sym-lib-table` to `~/.config/kicad/6.0/` instead of the active KiCad `9.0` configuration directory.
- **Fix:** Dynamically queried the installed `kicad` version via `dpkg-query` and copied `sym-lib-table` into `~/.config/kicad/$kicad_ver/`.

### 5. Virtual Environment Isolation for PyQt5/Matplotlib (`installDependency`)
- **Issue:** On Python 3.14 (Ubuntu 26.04), isolated `virtualenv` environments attempt to compile `PyQt5` from source instead of using system-installed `python3-pyqt5` and `python3-matplotlib` packages.
- **Fix:** Updated `virtualenv` creation to `virtualenv --system-site-packages $config_dir/env`.

---

## 🔧 NGHDL (`installNghdl` / `install-nghdl-26.04.sh`)

### 6. NGHDL OS Detection, GTK2 Module & LLVM 18 Compatibility
- **Issue:** `nghdl/install-nghdl.sh` exited with `Unsupported Ubuntu version: 26.04 ()`; `install-nghdl-24.04.sh` requested the removed `libcanberra-gtk-module` package and defaulted to LLVM 21/22, which is incompatible with `ghdl-4.1.0`.
- **Fix:** Patched `installNghdl()` in `install-eSim-26.04.sh` to generate `install-nghdl-26.04.sh`, replacing `llvm`/`llvm-dev` with `llvm-18`/`llvm-18-dev`, pointing `--with-llvm-config` to `/usr/bin/llvm-config-18`, removing `libcanberra-gtk-module`, and cleaning existing `$ghdl` folders before extraction.

### 7. GCC 15 C23 Standard `bool` Typedef Error in Ngspice (`installNGHDL`)
- **Issue:** Compiling `nghdl-simulator` (Ngspice) on Ubuntu 26.04 (GCC 15, which defaults to `-std=gnu23`) failed with:
  `error: ‘bool’ cannot be defined via ‘typedef’` (`‘bool’ is a keyword with ‘-std=c23’ onwards`).
- **Fix:** Passed `CFLAGS="-std=gnu17"` to `../configure` during the NGHDL simulator build.

---

## 🖥️ Runtime Fix (`createDesktopStartScript`)

### 8. Wayland Splash Screen Focus Freeze on First Launch
- **Issue:** Under Ubuntu 26.04's default Wayland session, the PyQt5 `WindowStaysOnTopHint` splash screen obscured the initial workspace dialog.
- **Fix:** Added `export QT_QPA_PLATFORM=xcb` to the generated `/usr/bin/esim` launcher script so eSim runs smoothly via XWayland.
