 
## Multiple Distro Install USB

### Creation via Linux
#### Setup environment variables
```
export mediamnt=<target mountpoint>
export efimntusb=<efi mountpoint> 
export datamntusb=<data mountpoint>
export devusb=<device>
export efipartusb=<efi partition>
export datapartusb=<data partition>
```

<details>
<summary>Example</summary>
```
export mediamnt=/media
export efimntusb=${mediamnt}/efi
export datamntusb=${mediamnt}/data
export devusb=/dev/sda
export efipartusb=${devusb}1
export datapartusb=${devusb}2
```
</details>

#### Create partitions
##### EFI Partition
```
sudo parted -s ${devusb} mklabel msdos
sudo parted -s ${devusb} mkpart primary 1MiB 1024MiB
```

##### Activate esp and boot flags on the USB
```
sudo parted -s ${devusb} set 1 esp on
sudo parted -s ${devusb} set 1 boot on
```

##### Format EFI partition
```
sudo mkfs.fat -F32 ${efipartusb}
```

---

##### Data Partition
```
sudo parted -s ${devusb} mkpart primary 1024MiB 100%
```

##### Format EFI partition
```
sudo mkfs.ext4 ${datapartusb}
```

#### Create mount points and mount partitions
```
# Create the mountpoints
#$ sudo mkdir /media/{efi,data}
$ sudo mkdir ${efimntusb}
$ sudo mkdir ${datamntusb}

# Mount the EFI partition
$ sudo mount ${efipartusb} ${efimntusb}

# Mount the data partition
$ sudo mount ${datapartusb} ${datamntusb}
```

#### Setup ISO Directory and Fetch Media
##### Setup ISO directory
```
sudo mkdir ${datamntusb}/boot/iso
chown 1000:1000 ${datamntusb}/boot/iso
```

##### Fetch Media
```
cd ${datamntusb}/boot/iso
wget <iso url>
```
<detail>
<summary>Example</summary>
```
wget https://releases.ubuntu.com/22.04.4/ubuntu-22.04.4-live-server-amd64.iso
```
</detail>

##### Create a general version link to the iso
This link should point to the latest downloaded version on the USB drive.
```
i.e.
cd ${datamntusb}/boot/iso
ln -s ubuntu-22.04.4-live-server-amd64.iso ubuntu-22.04-live-server-amd64.iso
```


#### Setup Grub Menu Entries
```
menuentry "Ubuntu 22.04 Live Server (AMD64)" {
    iso_path="/boot/iso/ubuntu-22.04-live-server-amd64.iso"
    loopback loop "${iso_path}"
    bootoptions="iso-scan/filename="${iso_path}" boot=casper quiet splash ---"
    linux (loop)/casper/vmlinuz ${bootoptions}
    initrd (loop)/casper/initrd
}
```
<detail>
<summary>Additional Examples</summary>
```
# Clonezilla
menuentry "Clonezilla live (Default settings, VGA 1024x768)" {
  iso_path="/boot/iso/<ISO file>"
  loopback loop "$iso_path"
  bootoptions="iso-scan/filename=${iso_path} boot=live union=overlay username=user config components quiet noswap edd=on nomodeset nodmraid locales= keyboard-layouts= ocs_live_run='ocs-live-general' ocs_live_extra_param='' ocs_live_batch=no vga=791 ip= net.ifnames=0  nosplash i915.blacklist=yes radeonhd.blacklist=yes nouveau.blacklist=yes vmwgfx.enable_fbdev=1"
  linux (loop)/live/vmlinuz $bootoptions
  initrd (loop)/live/initrd.img
}
```

```
# Rocky
menuentry "Rocky Linux Live Server" {
  iso_path="/boot/iso/<ISO file>"
  loopback loop "$iso_path"
  probe --label --set=cd_label (loop)
  bootoptions="iso-scan/filename=$iso_path root=live:CDLABEL=$cd_label rootfstype=auto ro rd.live.image quiet  rhgb rd.luks=0 rd.md=0 rd.dm=0"
  linux (loop)/isolinux/vmlinuz0 $bootoptions
  initrd (loop)/isolinux/initrd0.img
}
```
</detail>

#### References
Multiboot USB creation Tutorial

https://linuxconfig.org/how-to-create-multiboot-usb-with-linux

Example config generators

https://github.com/aguslr/multibootusb/tree/main/mbusb.d

Multiboot USB Creation tool

https://unetbootin.github.io/

 
