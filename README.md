Dualboot FreeBSD/Ubuntu
=======================

Install FreeBSD
---------------

https://bsd-hardware.info/?probe=5359b4dee9


### Version

15 & 14 work, 13 not tested.


### GPT partition table

| Type          | size  | mountpoint  |
|---------------|-------|-------------|
| `EFI`         | 260M  | `/boot/efi` |
| `freebsd-ufs` | X     | `/`         |
| `linux-data`  | X     |             |
| ...           |       |             |

Using ZFS for root works too, but requires an extra partition.


### WiFi

`iwlwifi` works, but unstable: connection issues, crashes. Ok now (2024).


### Bluetooth

Requires `iwmbt-firmware` package, tested with bluetooth headphones.


### Video

Install `drm-510-kmod` and `gpu-firmware-kmod` packages, add users to `video`
group. Backlight can be adjusted with `backlight` utility.


### Card reader

Not supported.


### Camera

Install `webcamd` and add users to `webcamd` group.


### Audio

Speakers are not automatically muted by when headphones are connected, can be
solved using device hints
(<https://forums.freebsd.org/threads/jack-detection-fails.48289/>)

Pins (`sysctl dev.hdac.1.pindump=1 && dmesg`):
```
hdaa1: nid   0x    as seq device       conn  jack    loc        color   misc
hdaa1: 18 90a60140 4  0  Mic           Fixed Digital Internal   Unknown 1
hdaa1: 20 90170110 1  0  Speaker       Fixed Analog  Internal   Unknown 1
hdaa1: 21 01211020 2  0  Headphones    Jack  1/8     Rear       Black   0
hdaa1: 23 40000000 0  0  Line-out      None  Unknown 0x00       Unknown 0
hdaa1: 24 01a11030 3  0  Mic           Jack  1/8     Rear       Black   0
hdaa1: 25 411111f0 15 0  Speaker       None  1/8     Rear       Black   1
hdaa1: 26 411111f0 15 0  Speaker       None  1/8     Rear       Black   1
hdaa1: 27 411111f0 15 0  Speaker       None  1/8     Rear       Black   1
hdaa1: 30 411111f0 15 0  Speaker       None  1/8     Rear       Black   1
```

Add `hint.hdaa.1.nid21.config="as=1 seq=15"` to `/boot/device.hints`: `as=1` --
move headphones to the same association as the speaker, `seq=15` -- override
speaker with headphones when connected, see `man snd_hda`.

Can also be changed manually: `sysctl hw.snd.default_unit=2`


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

It might be necessary to drop default loader from the boot order list.
```
sudo efibootmgr --delete-bootnum -b 0005
sudo efibootmgr --bootorder 0000
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

### Battery

```
sudo apt install upower
upower -i /org/freedesktop/UPower/devices/battery_BAT0
```

### Shared filesystems

- Mounting of UFS: `sudo mount -r -t ufs -o ufstype=ufs2 ...`

- Force import of zfs pools in Lunux by changing `ZPOOL_IMPORT_OPTS` in
  `/etc/default/zfs`.



Notes
=====

1. UID assignment policies are slightly different: FreeBSD starts with 1001,
   Linux -- 1000. This might lead to some minor issues if there are shared
   filesystems.

2. FreeBSD development branch generally includes newer version of ZFS than
   provided by Linux packages (`sudo apt install zfsutils-linux`), which may
   prevent access to ZFS systems created in FreeBSD from Linux. Use
   compatibility option to prevent this, e.g.
   `zpool create -m <mountpoint> -o compatibility=/usr/share/zfs/compatibility.d/zol-0.8 <pool> /dev/<partition>`
