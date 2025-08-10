+++
title = "Run Synology in QNAP NAS with PVE"
date = 2025-06-29T20:53:00+08:00
lastmod = 2025-08-10T18:44:06+08:00
tags = ["NAS"]
categories = ["Misc"]
draft = false
author = "KK"
nocomment = true
nodate = true
nopaging = true
noread = true
+++

Three years ago, I bought a QNAP TS-453Dmini NAS. Although it has a slow WEB UI and slow restart, it still fits my needs as all of the applications I need are running in Docker.

Recently, I want to move some files from my Mac to NAS to save space. I need a application behave like Dropbox, which can show all the files in the NAS and only download the files I need. I have tried the QSync, but it does not have thumbnails for cloud image and it does not have icons to show the file status. I also tried the [Seafile](https://www.seafile.com/home/), it's a powerful application, which requires 4G RAM to run, and there is bug in the thumbnail. I used to have a Synology ARM NAS, the Synology Drive has all the features I need, so I want to run it on my QNAP NAS. After some research, I managed to run Synology and QNAP together on my NAS. Here is the guide.


## Install PVE on QNAP {#install-pve-on-qnap}

To run Synology on QNAP, the first step is to install PVE. I put the PVE iso to a [Ventoy](https://www.ventoy.net/en/index.html) USB drive and reboot the NAS. When hearing the beep, press the `DEL` key immediately to enter the BIOS. Change the boot order to boot from the Ventoy and then install the PVE on the NAS. Remember to move the PVE disk to HDD bay 3 or 4, and the QNAP bios won't set the bay 1 or 2 as boot disk.


## Install Synology on PVE {#install-synology-on-pve}

It's pretty easy to install Synology on PVE, there are many guides in the Internet. Thanks to the [RR](https://github.com/RROrg/rr) project, you can install Synology on any x86 hardware.


## Install QNAP on PVE {#install-qnap-on-pve}

I have one SSD and one HDD. The SSD is used for PVE and Synology, but I don't have another disk to move the data to Synology. So I still need to run QNAP on PVE. Although the QNAP is not as popular as Synology, there is still a way to run it on PVE. I just followed [this guide](https://post.smzdm.com/p/adm3qxen/) to install QNAP on PVE. To pass-through the HDD, just run `ls -l /dev/disk/by-id/` to show all device and `` qm set {qnap vm id} -sata0 /dev/disk/by-id/{hdd_id}` `` to pass-through the HDD to QNAP VM. When restart the QNAP VM, it will ask you whether to load the OS from HDD, reset OS (but keep the data) or init the OS.

{{< figure src="/images/qnap_recover.png" class="image-size-m" >}}

However, when I choose load the OS, it reboot and back to the same screen. I guess it might be a compatibility issue. Reset OS may work. For me, I create a new disk for the QNAP VM an install the QNAP OS, then I pass-through the HDD to the QNAP VM. Then in the `Storage and Snapshot` app, under `Storage-Disks/VJOBD`, on the right corner, click the `More` button and choose `Recover`. I managed to recover the data in the HDD. After that, I can use the QNAP as usual.


## Synology OS vs QNAP OS {#synology-os-vs-qnap-os}

After using Synology for a while, I found the Synology OS is more user-friendly than QNAP OS. The UI is more responsive and the applications are more polished. There are more community applications available for Synology, and the overall experience feels more integrated. The Synology OS is more optimized. For example, the Synology backend services uses much less CPU and RAM than QNAP. I will migrate all of my data to Synology in the future once I got a new HDD.


## Ref: {#ref}

-   [除了 synology drive，还有支持选择性同步的本地部署云盘吗 ](https://www.v2ex.com/t/1135848)
-   [保姆级安装黑威联通](https://post.smzdm.com/p/adm3qxen/)
