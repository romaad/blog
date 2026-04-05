---
title: "Home assistant setup 1/N"
date: 2026-4-5
body: Setting up home assistant with libvirt
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
* 
