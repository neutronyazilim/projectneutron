# Project Neutron

**Project Neutron** is a hardware-focused, minimalist AOSP (Android Open Source Project) device tree and system integration monorepo maintained under **Neutron Yazılım**. Designed to be free from unnecessary system bloat, background telemetry, and heavy UI layers, its primary objective is to bring clean, modern Android builds to legacy flagship hardware with a low memory footprint while strictly adhering to open-source standards.

---

## Core Principles & Architecture

* **Bloat-Free & Vanilla First:** Closed-source Google services (GApps) and proprietary vendor bloatware are excluded by default, delivering a clean and responsive base system.
* **Opinionated Personal Changes:** Built around custom system tweaks, personal UI preferences, and workflow-oriented adjustments tailored for daily use.
* **Monorepo Organization:** Device trees, common vendor configurations, local manifests, and build automation scripts are versioned together under a single centralized repository.
* **Hardware Compatibility & Stability:** Stock Hardware Abstraction Layer (HAL) components, SELinux policies, and kernel configurations are kept clean, auditable, and maintainable.

---

## Personal Changes & Customizations

Project Neutron includes opinionated modifications and personal tweaks layered on top of the base AOSP/LineageOS tree:

* **System & UI Overrides (`overlay-neutron/`):** Custom status bar padding, QS tile layout tweaks, default accent/theme adjustments, and cleaner system defaults.
* **Debloated Product Configuration:** Stripped-down `PRODUCT_PACKAGES` list removing unused legacy AOSP apps and background services to maximize free RAM.
* **Custom Default Properties (`vendor.prop` / `system.prop`):** Tuned animation scales, memory management flags (LMKD), and audio/network defaults suited for personal daily driving.
* **Patchset & Feature Additions:** Custom framework/SystemUI patches and personal quality-of-life additions maintained directly inside the monorepo.

---

## Repository Structure

The repository follows a modular directory hierarchy designed for direct integration into an AOSP source tree:

~~~text
projectneutron/
├── device/
│   └── xiaomi/
│       └── chiron/               # Xiaomi Mi MIX 2 (chiron) device tree
│           ├── configs/          # Audio, media, GPS, and thermal configuration files
│           ├── overlay/          # AOSP system and framework resource overlays
│           ├── overlay-neutron/  # Project Neutron-specific UI and system overrides
│           ├── sepolicy/         # Device-specific SELinux security policies
│           ├── ultrasound/       # Elliptic Labs ultrasonic proximity sensor configuration
│           ├── AndroidProducts.mk
│           ├── BoardConfig.mk    # SoC, partition layout, kernel, and hardware flags
│           ├── device.mk         # Hardware packages, HALs, and system properties
│           ├── neutron_chiron.mk # Product definition (PRODUCT_NAME, brand, model)
│           └── proprietary-files.txt # Vendor binary blob list
├── vendor/
│   └── neutron/                  # Common build rules, overlays & personal customizations
│       └── config/
│           └── common_full_phone.mk
├── manifests/
│   └── chiron.xml                # Local manifest for source synchronization
├── scripts/
│   └── build.sh                  # Environment setup and automated build helper
├── LICENSE                       # Apache License 2.0
└── README.md
~~~

---

## Supported Devices & Hardware Matrix

| Device | Codename | SoC | Memory / Storage | Kernel | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Xiaomi Mi MIX 2** | `chiron` | Qualcomm MSM8998 (Snapdragon 835) | 6 GB LPDDR4X / UFS 2.1 | Linux 4.4 (`msm8998`) | **Bring-up / WIP** |

### `chiron` Hardware Notes
* **Display:** 5.99" IPS LCD, 1080x2160 (18:9), `TARGET_SCREEN_DENSITY := 400`
* **Audio & Sensors:** Ultrasonic proximity sensor support preserved via `AUDIO_FEATURE_ELLIPTIC_ULTRASOUND_SUPPORT := true`.
* **Partitioning:** Classic A-only partition layout (`system`, `vendor`, `boot`, `recovery`).

---

## Setup & Source Synchronization

Follow the steps below to integrate Project Neutron device configurations into an existing AOSP or LineageOS-based source tree.

### 1. Prerequisites
Ensure your build environment meets the following system requirements:
* **OS:** Up-to-date Linux distribution (Arch Linux, Ubuntu 22.04/24.04 LTS, etc.)
* **Hardware:** Minimum 16 GB RAM (32 GB or zram recommended) and ~300 GB free NVMe/SSD storage
* **Dependencies:** `git`, `repo`, `bc`, `bison`, `build-essential`, `ccache`, `curl`, `flex`, `g++-multilib`, `gcc-multilib`, `gnupg`, `gperf`, `imagemagick`, `libncurses-dev`, `libssl-dev`, `libxml2-utils`, `lzop`, `pngcrush`, `rsync`, `schedtool`, `squashfs-tools`, `xsltproc`, `zip`, `zlib1g-dev`

### 2. Local Manifest Integration
Create the `.repo/local_manifests` directory at the root of your AOSP workspace and copy the local manifest from this monorepo:

~~~bash
mkdir -p .repo/local_manifests
cp /path/to/projectneutron/manifests/chiron.xml .repo/local_manifests/neutron_chiron.xml
~~~

### 3. Sync Sources
Run `repo sync` to fetch the device tree, common `msm8998-common` dependencies, and kernel sources:

~~~bash
repo sync -c --no-clone-bundle --no-tags --optimized-fetch -j$(nproc)
~~~

---

## Building

Once the source tree is synced, configure `ccache` and initiate the build process.

### 1. Configure Build Cache (Optional)
Significantly reduce incremental build times by enabling `ccache`:

~~~bash
export USE_CCACHE=1
export CCACHE_EXEC=/usr/bin/ccache
ccache -M 50G
~~~

### 2. Initialize Environment & Select Target
~~~bash
source build/envsetup.sh
lunch neutron_chiron-userdebug
~~~

### 3. Build Flashable Package
Compile the full installation ZIP package:

~~~bash
mka bacon -j$(nproc)
~~~

Once completed, the output package will be generated under `out/target/product/chiron/`.

---

## Contributing

Bug reports, hardware patches, and configuration improvements are welcome via Pull Requests:

1. Fork the repository or create a feature branch (`git checkout -b fix/chiron-audio`).
2. Ensure changes adhere to standard AOSP Makefile and SELinux policy conventions.
3. Keep commit messages clear and descriptive (e.g., `chiron: overlay: adjust statusbar padding`).
4. Open a Pull Request detailing your changes.

---

## Credits & Acknowledgments

* **[LineageOS](https://lineageos.org/):** Base `msm8998-common` and `android_device_xiaomi_chiron` hardware configurations.
* **[AOSP](https://source.android.com/):** Core Android Open Source Project platform.
* **Xiaomi MSM8998 Developer Community:** Kernel and vendor blob compatibility work.

---

## License

This project is licensed under the **Apache License 2.0** — see the [LICENSE](LICENSE) file for details.
