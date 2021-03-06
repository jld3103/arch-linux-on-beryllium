# Arch Linux on beryllium (Xiaomi Poco F1)

My efforts to run Arch Linux natively (no [Halium](https://halium.org)) on the Poco F1.  
I only created this script (yet), but the most effort comes from the people working
on https://gitlab.com/sdm845-mainline/sdm845-linux.  
Currently, you can only boot from the SD card or using QEMU, but support for internal memory will be coming.

# Building

To speed up package downloads you can use the `LOCAL_MIRROR` environment variable with and value
like `http://192.168.178.30:8080/\$repo/\$arch` with https://github.com/arch-beryllium/mirror as local mirror tool.  
(If you have used `arch-repo-mirror` previously for mirroring there is a quick migration guide available in the new
repo).  
Without it, you will probably wait for hours on every build, so it's highly recommended.

```bash
./build.sh barebone
./build.sh phosh
./build.sh plasma-mobile
./build.sh lomiri
```

To include the default apps (probably what you want for non development purposes):

```bash
./build.sh phosh-apps
./build.sh plasma-mobile-apps
./build.sh lomiri-apps
```

To only update the kernel, initramfs and boot.img:

```bash
./build.sh barebone-bootimg
./build.sh phosh-bootimg
./build.sh plasma-mobile-bootimg
```

# Flashing onto SD card

You need at least an 8 GB SD card, but more is better.  
Everything on the SD card will be deleted so watch out what you are doing.

```bash
./flash_sdcard.sh barebone /dev/sdX
./flash_sdcard.sh phosh /dev/sdX
./flash_sdcard.sh plasma-mobile /dev/sdX
./flash_sdcard.sh lomiri /dev/sdX
```

# Booting

Put the SD card into the device.  
Boot the device to bootloader mode (you might need to reboot it once to bootloader mode if you put the SD card into the
device after it already booted to bootloader mode).

To temporarily boot from the SD card use:

```bash
./boot.sh barebone
./boot.sh phosh
./boot.sh plasma-mobile
./boot.sh lomiri
```

To permanently boot from the SD card use:

```bash
./flash_boot.sh barebone
./flash_boot.sh phosh
./flash_boot.sh plasma-mobile
./flash_boot.sh lomiri
```

On the first boot it will take a longer time, because it resizes the rootfs to the full size of the SD card. Please
don't turn it off in that time.

# SSHing into the device

Run this script to ssh into the device:

```bash
./ssh.sh
```

The default password is `123456`.

# Enable USB internet forwarding

This will block Wi-Fi on the device until rebooting.  
Host side:

```bash
./enable_forwarding.sh
```

Device side:

```bash
sudo route add default gw 10.15.19.100
```

# QEMU

You can also run the image in QEMU:

```bash
./qemu.sh barebone
./qemu.sh phosh
./qemu.sh plasma-mobile
./qemu.sh lomiri
```

It uses a qcow2 overlay image, so the rootfs won't be changed.  
It also automatically forwards network requests, and you don't need to use ssh to get a shell, because script
automatically opens a console for you.

# Kernel updates

## Device

When you flash the boot.img permanently, every kernel update on the device will flash the new boot.img onto the boot
partition that is generated.  
If you booted temporarily from a boot.img then the new boot.img won't be flashed onto the boot partition.  
To still use the new kernel, update the boot.img with the instructions from the QEMU section below.

## QEMU

The kernel update that is installed in the image won't be used, so you have to _build_ (actually just extracts it from
the image) it manually:

```bash
./build.sh barebone-bootimg
./build.sh phosh-bootimg
./build.sh plasma-mobile-bootimg
./build.sh lomiri-bootimg
```

# Anbox

Install and setup Anbox:

```bash
sudo pacman -S anbox anbox-image-aarch64 android-tools
sudo systemctl enable --now anbox-container-manager
systemctl enable --now --user anbox-session-manager
```

and then reboot.

# Cross-compiling packages

If you want to cross-compile packages you can use the `makepkg.sh` script.  
You can pass all flags to the script that makepkg accepts.

## Cross-compiling the kernel packages

```bash
git clone https://github.com/arch-beryllium/linux-beryllium
cd linux-beryllium
/path/to/this/repo/makepkg.sh
```