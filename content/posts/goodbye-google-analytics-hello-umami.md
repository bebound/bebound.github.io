+++
title = "Goodbye Google Analytics, Hello Umami"
date = 2026-11-16T20:54:00+08:00
tags = ["Umami"]
categories = ["Misc"]
draft = false
author = "KK"
nocomment = true
nodate = true
nopaging = true
noread = true
+++

Recently, my open-source project [AltGallery](https://github.com/bebound/AltGallery) was recommended by a popular telegram channel [iOS宝藏](https://t.me/iosrxwy), and picked up lots of new stars on GitHub. I was glad to see it. Because of this, I also add a few things: a Chinese readme, [AltGallery Web](https://altgallery.fromkk.com) and Chinese translation. I was also curioushow many people actualy use it. GitHub Traffic only shows rough numbers like this; I wanted something more accurate.

{{< figure src="/images/umami_github.png" >}}

My blog still runs on Google Analytics, and honestly, I don’t like it. It’s complicated and slow, and it can’t even filter out bot traffic.

|                                |                                |
|--------------------------------|--------------------------------|
| ![](/images/umami_google1.png) | ![](/images/umami_google2.png) |

After some research, I decided to switch to [Umami](https://umami.is/) to replace Google Analytics. Umami offers a [hobby plan](https://umami.is/pricing) that tracks 100k events for 1 website for free. You can also [run it on Vercel](https://docs.umami.is/docs/guides/running-on-vercel) to cover more websites and events. I have tried it, but the network speed is insanely slow. (Netlify is fast in my experience, you may want to try [Netlify guide](https://docs.umami.is/docs/guides/running-on-netlify) with a [free Neon database](https://neon.com/) instead.) So I decided to deploy it on my server. The setup is simple: run Umami and Postgres with docker-compose and you’re good to go.


## Umami usage {#umami-usage}

To get started, log in to the dashboard and create a website, then copy the tracking code onto your site. Some ad blockers may block the tracking code, so you can use the [custom tracker script name](https://docs.umami.is/docs/environment-variables#tracker_script_name) to get around them. And now you should be able to see the traffic data in the dashboard. Here are some useful features of Umami:


### Track Event {#track-event}

If you want to track some specific events, you can use the [track event](https://docs.umami.is/docs/track-events) function. The design is elegant: you can use the `data-attributes` or JavaScript code. Here’s how I track clicks on the Add altstore source button on AltGallery Web:

```html
<a class="btn-mini"
  data-umami-event="Add single source"
  data-umami-event-app="HermesEnabler">Add
</a>
```

This will track the event as `Add single source` with the property `app=HermesEnabler`.


### Link Tracking {#link-tracking}

Umami can also create a [tracking link](https://docs.umami.is/docs/links). That’s handy when you want to track clicks on a link.


### Pixel Tracking {#pixel-tracking}

[Pixel tracking](https://docs.umami.is/docs/pixels) is also supported in Umami. I wanted to use it to track traffic to the AltGallery repo, but GitHub doesn’t allow that.

> To host your images, GitHub uses the open-source project Camo. Camo generates an anonymous URL proxy for each file which hides your browser details and related information from other users. On GitHub.com, the URL starts <https://<subdomain>.githubusercontent.com/>, with different subdomains depending on how you uploaded the image.
>
> -- <https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/about-anonymized-urls>

---

PS: Umami is the third app which uses Postgres, alongside [Tiny Tiny RSS](https://tt-rss.org/) and [Linkwarden](https://linkwarden.app/). Previously, They use two PG16 instances. I don't want to waste the resource to spin up a third. So I just use Claude to help me migrate the data to a new PG 18 instance, and update the docker compose file. It did a good job. I really couldn't imagine that AI can be so smart like this two years ago.


## Ref {#ref}

1.  [Environment Variables - Umami Docs](https://docs.umami.is/docs/environment-variables)
