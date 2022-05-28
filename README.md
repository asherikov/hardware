Dualboot FreeBSD/Ubuntu
=======================

Install FreeBSD
---------------

https://bsd-hardware.info/?probe=5359b4dee9


### Version

14 works, 13 might.


### GPT partition table

| Type          | size  | mountpoint  |
|---------------|-------|-------------|
| `EFI`         | 260M  | `/boot/efi` |
| `freebsd-ufs` | X     | `/`         |
| `linux-data`  | X     |             |
| ...           |       |             |

Using ZFS for root works too, but requires an extra partition.


### WiFi

`iwlwifi` works, but unstable: connection issues, crashes.


### Bluetooth

Requires `iwmbt-firmware` package, tested with bluetooth headphones.


### Video

Install `drm-510-kmod` and `gpu-firmware-kmod` packages, add users to `video`
group. Backlight can be adjusted with `backlight` utility.


### Card reader

Not supported.


### Camera

Install `webcamd` and add users to `webcamd` group.


### Other

- `amdtemp` thermal sensor driver



Install Ubuntu
--------------

https://linux-hardware.org/?view=computers&type=Notebook&vendor=TUXEDO&model_like=Pulse


### Version

>= 20.04



### Boot manager

Install `rEFInd` boot manager, http://www.rodsbooks.com/refind/installing.html,
it should detect both FreeBSD and Linux and boot either of them.

```
apt-add-repository ppa:rodsmith/refind
apt update
apt install refind
```


### Tuxedo stuff

```
wget -O - http://deb.tuxedocomputers.com/0x54840598.pub.asc | sudo apt-key add -
sudo apt-key adv --fingerprint 54840598
apt install -y tuxedo-keyboard
```


### Video

Change Backlight manually

```
echo 50 | sudo tee /sys/class/backlight/amdgpu_bl0/brightness
```
or using a `brightnessctl`
```
sudo apt install brightnessctl brightness-udev
sudo brightnessctl set 50
```


Notes
=====

1. UID assignment policies are slightly different: FreeBSD starts with 1001,
   Linux -- 1000. This might lead to some minor issues if there are shared
   filesystems.

2. Development branch of FreeBSD generally includes newer version of ZFS than
   provided by Linux packages. This may prevent access to ZFS systems created
   in FreeBSD from Linux, e.g., `feature@head_errlog` feature (check with
   `zpool get all data`).
   sudo apt install zfsutils-linux
