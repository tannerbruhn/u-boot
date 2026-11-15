# Working on this U-Boot fork

This fork boots postmarketOS on the Samsung Galaxy S10 (SM-G973F, `beyond1lte`, Exynos 9820) and supplies the
device tree that Linux runs with. Humans and AI coding agents are both welcome to write code here. Upstream U-Boot's
contribution rules apply when sending patches to the U-Boot list; follow them then.

## Where the S10 work is
- Device tree: `dts/upstream/src/arm64/exynos/exynos9820.dtsi`, `exynos9820-beyond1lte.dts`,
  `exynos9820-tmu.dtsi`: Wi-Fi PCIe nodes, the reserved bootloader debug-log area, the Mali-G76 GPU node with its
  rates (156-702 MHz) and the g3d thermal trip.
- Kernel side: https://github.com/tannerbruhn/exynos-9820-mainline-linux (install notes in S10-WIFI.md).

## Build
```sh
make exynos9820_defconfig && make -j"$(nproc)"
```
Package `u-boot.bin` as an Android boot image, header v0, kernel only, with the same header fields as the
release's `uboot-wifi-boot.img` (check by re-packing the release's own u-boot.bin: the result must be
byte-identical).

## Rules
1. **Check device-tree changes before flashing.** Compile the DTB and compare it, sorted (`dtc -s`), against the
   device tree the phone runs (`/sys/firmware/fdt`). Every difference should be one you meant.
2. **Flash only through the release installer path:** it backs up the boot partition first and verifies the
   write. Keep a copy of the backup off the phone. If U-Boot does not start, recovery needs a PC and Samsung's
   download mode.
3. **Hardware values come from Samsung's vendor device tree or GPL kernel.** GPU rates only from the vendor DVFS
   table: the firmware (ACPM) sets the voltage per rate and rejects anything else. Never add `opp-microvolt` for
   rails the firmware owns.
4. **Memory:** any region the bootloader or firmware writes after boot must be reserved. v1 shipped a corruption
   bug because one was not (fixed in v2). Changes to memory or reserved regions need a stress test under memory
   pressure on the phone.
5. **Stock kernels must keep booting.** The stock postmarketOS kernel also boots with this device tree; new nodes
   must be harmless for it (it has no Panfrost, for example, so it ignores the GPU node).
6. **Keep authorship** of adapted work (the GPU node is adapted from Gregor Sprint's work in
   omnipha/linux-samsung-s10-exynos9820). Commits in upstream style, as
   `tannerbruhn <tannerbruhn@users.noreply.github.com>`, without co-author or tool trailers.
