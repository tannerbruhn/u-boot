> **Galaxy S10 (SM-G973F) fork.** The `main` branch adds the S10 device tree pieces for postmarketOS on the
> mainline Exynos 9820 port: PCIe Wi-Fi, the reserved bootloader debug-log area, and the Mali-G76 GPU node (adapted
> from Gregor Sprint's work in omnipha/linux-samsung-s10-exynos9820) with the GPU rates up to the stock 702 MHz.
> This S10 work was written by AI and tested on the phone. Install notes: S10-WIFI.md in the kernel fork.

> **The S10 project.**
>
> These four repos turn an old Samsung Galaxy S10 (SM-G973F, Exynos 9820) into a small always-on LLM server under
> postmarketOS. The kernel and U-Boot forks are general S10 support; the other two are the LLM side.
>
> - [exynos-9820-mainline-linux](https://github.com/tannerbruhn/exynos-9820-mainline-linux): kernel (Wi-Fi, charger,
>   Mali-G76 GPU through Panfrost)
> - [u-boot](https://github.com/tannerbruhn/u-boot): bootloader and device tree (Wi-Fi, reserved memory, GPU node and
>   clocks)
> - [llama.cpp-s10](https://github.com/tannerbruhn/llama.cpp-s10): inference (ternary CPU kernels, Mali GPU assist)
> - [s10-llm-recipes](https://github.com/tannerbruhn/s10-llm-recipes): models, quantization, server setup, measured
>   numbers
>
> Updates: [@t24_lab on X](https://x.com/t24_lab)

# Installation (Ubuntu 26.04 Host)

## Dependencies
```bash
sudo apt update
sudo apt install gcc-aarch64-linux-gnu adb mkbootimg git \
  bc bison build-essential coccinelle \
  device-tree-compiler dfu-util efitools flex gdisk graphviz imagemagick \
  libgnutls28-dev libguestfs-tools libncurses-dev \
  libpython3-dev libsdl2-dev libssl-dev lz4 lzma lzma-alone openssl \
  pkg-config python3 python3-asteval python3-coverage python3-filelock \
  python3-pkg-resources python3-pycryptodome python3-pyelftools \
  python3-pytest python3-pytest-xdist python3-sphinxcontrib.apidoc \
  python3-sphinx-rtd-theme python3-subunit python3-testtools \
  python3-venv swig uuid-dev
```

## Compilation and Installation
```bash
git clone https://github.com/chiffathefox/u-boot -b exynos9820
cd u-boot
make CC=aarch64-linux-gnu-gcc CROSS_COMPILE=aarch64-linux-gnu- exynos9820_defconfig
make CC=aarch64-linux-gnu-gcc CROSS_COMPILE=aarch64-linux-gnu- -j "$(nproc)"
mkbootimg --kernel u-boot.bin -o boot.img
adb wait-for-recovery
adb push boot.img /tmp
adb shell dd if=/tmp/boot.img of=/dev/block/platform/13d60000.ufs/by-name/boot
```

