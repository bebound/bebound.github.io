+++
title = "QNAP TS-453Dmini Review"
date = 2022-01-19T00:17:00+08:00
lastmod = 2025-07-18T19:07:22+08:00
tags = ["NAS"]
categories = ["Misc"]
draft = false
author = "KK"
nocomment = true
nodate = true
nopaging = true
noread = true
+++

My first NAS is Synology DS120j, which is ARM based entry level product. It's okay to use it for downloading and backup, but not power enough for running docker and virtual machine.

So I bought this NAS last month, and I'm satisfied with it. Here are the advantages and disadvantages.


## Advantages {#advantages}

1.  High performance.

    It is equipped with J4125 quad-core 2.0 GHz processor, 8G RAM, two 2.5G Ports and 4 bays. Here is the [spec](https://www.qnap.com/zh-cn/product/ts-453dmini/specs/hardware). Although J4125 is not the fastest CPU in 2022(the newer model coming with N5105), it is still able to run several docker containers together, and I can even run Synology and Windows 10 inside build-in `Virtualization Station`.

2.  Low Price.

    I bought is at 2050 Yuan (about $320). Synology has similar model DS920+, which is 70% more expensive.

3.  [Qtier](https://www.qnap.com/solution/qtier/en/)

    A special feature only available on QNAP. It will automatically move hot and cold data between SSD and HDD to get higher performance. There is one noticeable change: this NAS is so quiet and you can hardly hear HDD running.


## Disadvantages {#disadvantages}

1.  No native HEIC Support

    This means you can't preview the photos imported from iPhone. You have to pay for $12 to buy [CAYIN MediaSign Player](https://www.qnap.com/en-in/software/cayin-mediasign-player) to fix this.

2.  Slow Start and Shutdown

    I know it's rarely to restart NAS, but taking several minutes to restart is not acceptable. I hope QNAP engineers can improve this in the future.


## Docker Images Recommendation {#docker-images-recommendation}

Here are some docker images I think is useful:

-   `johngong/qbittorrent`

A torrent client which can also download new file through RSS feed.

-   `p3terx/aria2-pro`

Everyone knows what is `aria2`.

-   `dreamacro/clash`

A rule based proxy client.

-   `jeessy/ddns-go`

A simple DDNS client.

-   `linuxserver/plex`

Plex is app which helps you to manage and browser your media library. It can grab the metadata of TV shows, movies and music from Internet. It's provide app in different platform so you can access your media from anywhere.

There is only one thing I think that needs to improve: playback speed is fixed, which is not convenient when watching animation.

You can also give [Emby](https://emby.media) or [Jellyfin](https://jellyfin.org) a shot.

-   `portainer/portainer-ce`

QNAP has build-in `docker-compose` command. You can use this Web app if you prefer GUI.

-   `vaultwarden/server`

I deploy this on my VPS instead of NAS as port 443 is forbidden on NAS. It's a alternative of [1Password](https://1password.com). Although the app UI is not perfect, it has all the function required by a password manager. There is no official way to backup data, so I use `crontab` to run backup script to save data to Google Drive by [Rclone](https://rclone.org).

The backup script is quite simple, you need to link your Google Drive account as `google` in Rclone before using this.

```bash
#!/bin/sh
pwd
echo backing up
rm bit.zip
zip -rq bit.zip ./data -x ./data/icon_cache/* ./data/bitwarden.log
rclone copy bit.zip google:/应用/vaultwarden
```
