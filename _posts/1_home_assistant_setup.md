---
title: "Home assistant setup 1/N"
date: 2026-4-5
body: Setting up home assistant with libvirt
headerImage: false
tag:
- home-assistant
- home
- assistant
- setup
category: blog
author: romaad
---

To start my home automation journey, had multiple [options](https://www.home-assistant.io/getting-started/) for how it can be setup. 
The easiesnt one is of course containerised HA however it lacks certain features specifically apps like let's encrypt, MQTT and others which are essential for what I wanted (more on this later).

I wanted my home assistant OS (haos) machine to be used as a home-server as well, so didn't like the Pis and the customised HA boards, server racks were too powerful
and ODroid was also a bit expensive for its performance. So I bought an old desktop (OptiPlex 3070) which had all I needed: 500GB of SSD, 8GB of Ram and intel video acceleration and small form-factor!
To start I wanted a linux system that is good as a server system (no-UI) leaned to use ubuntu out of familiarity but everyone praised pure debian so I went for it.

## Debian installation
Installation steps are straightforward:
* Download a suitable image: https://www.debian.org/distrib/ (I went for full image as my interenet was shaky)
* You can either use one of the tools in the official doc on [how to write this to a USB](https://www.debian.org/CD/faq/#write-usb). However I just followed the copy-block way from [askubunutu](https://askubuntu.com/questions/1398432/how-to-burn-an-iso-file-to-a-usb):
```bash
$ sudo dd if=<input_file> of=<device_name>
$ sync
```
* Plug-in your usb in the machine, connect it to a screen and a keyboard and follow the steps (in some cases you might want to [enable usb-booting](https://superuser.com/questions/507111/if-usb-is-not-listed-in-bios-as-a-boot-option-does-that-mean-the-machine-cant)).
* Once your debian is installed, add an admin user and a personal user, preferably a home-assistant user as well to give haos a controlled access to machine resources.
* Connect to wifi if you haven't done so. Allocate a static ip on your router for the machine.
* We'll treat this as a server, even if you chose to have a debian desktop package, we don't need it connected to a screen and keyboard anymore. For the rest of the tutorial I used ssh to control my system.

## kvm installation
* Plenty of resources on the web, for debian I used the [official guide](https://wiki.debian.org/KVM)
* The important step that is not mentioned in any HA guides it to install a network bridge, it is explained in the debian KVM guide. But to summarise:
### Network bridge
* Check the temporary part of the [wiki](https://wiki.debian.org/BridgeNetworkConnections). You can also test with it but it will go away with restart.
* After those steps are done (the ones before the permenant config changes) check your bridge status with `sudo ip addr show | grep br`
* Make the config permenant:
  * Add the following to your `/etc/network/interfaces`
  ```conf
  auto virtbr0
  iface virtbr0 inet dhcp
      bridge_ports enp1s0
      address 192.168.1.10
      broadcast 192.168.1.255
      netmask 255.255.255.0
      gateway 192.168.1.1
  ```
  Where enp1s0 in this case is the Lan port, virtbr0 is your bridge name. I set a static address here `192.168.1.10`
  for the bridge so I can easilly get my virtual images addresses. You can remove that part to allow for dhcp allocation.
* After that we download the .qcow2 image from the HA [guide](https://www.home-assistant.io/installation/linux).
* For me I wanted to make my HA store some logs and media so I needed to extend a bit the original allocated disk for the image:
```bash
// Source - https://stackoverflow.com/a/38081468
// Posted by user2051965
// Retrieved 2026-04-06, License - CC BY-SA 3.0

cp small_image.qcow2 large_image.qcow2
qemu-img resize large_image.qcow2 +40G
# check large image boots
rm small small_image.qcow2
```
* To add the image to kvm images, I use the provided command from the mentioned guide (notice I add the bridge to it):
```bash
virt-install --name haos --description "Home Assistant OS" --os-variant=generic --ram=4096 \
 --vcpus=2 --disk <PATH TO QCOW2 FILE>,bus=scsi --controller type=scsi,model=virtio-scsi \
--import --graphics none --boot uefi --network bridge=<bridge_name>
```
* It will run for a while, use `virsh list` to make sure it is running.
* Follow with the onboarding https://www.home-assistant.io/getting-started/onboarding/ in the setup we put a virtual bridge at `192.168.1.10`,
you can access the frontend at http://192.168.1.10:8123
