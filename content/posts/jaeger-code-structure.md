+++
title = "Jaeger Code Structure"
date = 2019-09-22T17:07:00+08:00
lastmod = 2025-08-10T18:44:06+08:00
tags = ["Jaeger"]
categories = ["Misc"]
draft = false
author = "KK"
nocomment = true
nodate = true
nopaging = true
noread = true
+++

Here is the main logic for jaeger agent and jaeger collector. (Based on [jaeger](https://github.com/jaegertracing/jaeger) 1.13.1)

{{< figure src="/images/jaeger.svg" width="600" >}}


## Jaeger Agent {#jaeger-agent}

Collect UDP packet from 6831 port, convert it to `model.Span`, send to collector by gRPC


## Jaeger Collector {#jaeger-collector}

Process gRPC or process packet from Zipkin(port 9411).


## Jaeger Query {#jaeger-query}

Listen gRPC and HTTP request from 16686.
