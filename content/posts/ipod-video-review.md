+++
title = "iPod Video Review"
author = ["KK"]
date = 2023-12-26T11:16:00+08:00
lastmod = 2023-12-26T11:17:32+08:00
tags = ["iPod"]
draft = false
noauthor = true
nocomment = true
nodate = true
nopaging = true
noread = true
+++

I bought a iPod Video 5.5th Gen 80G recently. It's only 200 Yuan (about $30) and I'm satisfied with it.


## Rockbox {#rockbox}

The original firmware supports few audio format, it even can't play FLAC. I install rockbox on it, which support FLAC and other format and I can transfer music without using iTunes or Finder. It also support theme and plugin, which makes it more powerful.


### MacPod error {#macpod-error}

If you restore the iPod on macOS, it raises `Warning: This is a MacPod, Rockbox only runs on WinPods. See http://www.rockbox.org/wiki/IpodConversionToFAT32` during installation. The easiest way to fix this is to restore it on Windows.


### Permission denied error {#permission-denied-error}

When I tried to install rockbox 1.5.1 on macOS, it raised `could not open ipod permission denied` when I clicked install. Using `sudo /Applications/RockboxUtility.app/Contents/MacOS/RockboxUtility` can fix this issue. Some said using 1.4.1 on other OS can fix this issue, but I haven't tried it.


### Hangs on `Waiting for system to remount player` {#hangs-on-waiting-for-system-to-remount-player}

My iPod hangs on `Waiting for system to remount player` when I install rockbox. After timeout, I disconnected iPod and restart again. The startup screen shows `Can't load rockbox.ipod: file not found`. I connect iPod to computer and use rockbox utility to install rockbox again. I unselected the `bootloader`, and only install `rockbox`, `fonts` and `Plugin Data`. The error is gone.


### Theme {#theme}

There are many themes in rockbox. I prefer [fresh os light](https://themes.rockbox.org/index.php?themeid=3133&target=ipodvideo) and [adwaitapod Simplified](https://themes.rockbox.org/index.php?themeid=3407&target=ipodvideo). They also provide the dark version.


### Custom font {#custom-font}

See <https://d00k.net/wiki/rockbox_advanced/font_combining/>


## Replace SSD {#replace-ssd}

The original HDD is small, slow and fragile comparing with SSD, you can replace it with a SSD.


### Different Adapters {#different-adapters}

-   CE to m2 adapter (chip: JMB20330) and a 2242 m.2 SATA SSD (Recommended)
-   CE/SIF SSD (The product discontinued)
-   CE to TF card adaptor


### SSD Size {#ssd-size}

Not all iPod OS can support large SSD. It has the maximum track limit and SSD size limit in default OS. The track limitation stems from the RAM size, the large capacity model comes with more RAM and higher track threshold. For IPC 6th and 6.5th, if you release a SSD which is larger than 128G, iTunes only recongnize 128G. This is due to the LBA28 Limitation. Both Limitations can be eliminated by rockbox. If you want to stay in the original OS, I recommend you buying a 5.5th Gen 80GB or 7th Gen.

| Model Description | Model No.                             | iTunes Storage Limit (see note below)                                                   |
|-------------------|---------------------------------------|-----------------------------------------------------------------------------------------|
| 5th Gen 30Gb      | MA002 / MA146 / PA002 / PA146         | ~20000 Tracks                                                                           |
| 5th Gen 60Gb      | MA003 / MA147 / PA003 / PA147         | ~50000 Tracks                                                                           |
| 5.5th Gen 30Gb    | MA444 / MA446 / PA444 / PA446 / MA664 | ~20000 Tracks                                                                           |
| 5.5th Gen 80Gb    | MA448 / MA450 / PA448 / PA450         | ~50000 Tracks                                                                           |
| 6th Gen 80Gb      | MB029 / MB147 / PB029                 | 128Gb / ~50000 Tracks                                                                   |
| 6th Gen 160Gb     | MB145 / MB150 / PB145 / PB150         | 128Gb / ~50000 Tracks / **[Requires Ribbon](https://www.iflash.xyz/store/hdd-ribbon/)** |
| 6.5th Gen 120Gb   | MB565 / PB565 / MB562 / PB562         | 128Gb / ~50000 Tracks                                                                   |
| 7th Gen 160Gb     | PC297 / MC297 / PC293 / MC293         | ~50000 Tracks                                                                           |

Source: <https://www.iflash.xyz/store/iflash-compatibility/>


### Guide {#guide}

[iPod 5th Generation (Video) Hard Drive Replacement](https://www.ifixit.com/Guide/iPod+5th+Generation+(Video)+Hard+Drive+Replacement/607)


## Replace battery {#replace-battery}

iPod comes with a [580 mah](https://www.ifixit.com/products/ipod-classic-thin-replacement-battery?pk_vid=fb5c5f8766d880f516955682907df708) or [850 mah](https://www.ifixit.com/products/ipod-classic-thick-replacement-battery?pk_vid=fb5c5f8766d880f516955683667df708) battery base on the thin or thick back cover of your iPod.

After replace the SSD, there is a bigger space for battery. You can replace a larger battery to get longer battery life. Here is the guide: <https://zh.ifixit.com/Guide/iPod++Classic+%E7%94%B5%E6%B1%A0%E6%9B%B4%E6%8D%A2/561>.

Some seller even provide a 3000 mah battery, I'm not sure whether your iPod has enough space for it, please as the seller before buying it. Some said the 3000 mah battery is fake, it's actually a 1800 mah battery. Source: [3rd party
extended Battery guide](https://www.iflash.xyz/3rd-party-extended-battery-guide/)


### Guide {#guide}

[iPod 5th Generation (Video) Hard Drive Replacement](https://www.ifixit.com/Guide/iPod+5th+Generation+(Video)+Hard+Drive+Replacement/607)


## Ref {#ref}

-   [Topic: iPod classic 6th Gen LBA28 how to exceed 128gb limit?](https://forums.rockbox.org/index.php?topic=52281.0)
-   [SELLTOONE 128GB SSD for iPod Classic 6th 7th iPod Video 5Gen 5.5th Replace HS081HA MK8010GAH MK8022GAA MK1634GAL MK1231GAL ZIF CE Solid State Drive (128GB)](https://www.amazon.com/Classic-MK8010GAH-MK8022GAA-MK1634GAL-MK1231GAL/dp/B085F2XB1W#immersive-view_1695293331435)
-   [iPod 6/6.5 gen iFlash and Rockbox limitations?](https://www.reddit.com/r/IpodClassic/comments/15ei366/ipod_665_gen_iflash_and_rockbox_limitations/)
