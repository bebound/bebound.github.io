+++
title = "Timezone in JVM"
author = ["KK"]
date = 2020-10-18T23:49:00+08:00
lastmod = 2023-12-18T21:38:38+08:00
tags = ["Java", "Scala"]
draft = false
noauthor = true
nocomment = true
nodate = true
nopaging = true
noread = true
+++

I wrote a Scala code to get the current time. However, the output is different on the development server and docker.

```scala
import java.util.Calendar

println(Calendar.getInstance().getTime)
```

On my development server, it outputs `Sun Oct 18 18:01:01 CST 2020`, but in docker, it print a UTC time.

I guess it related to the timezone setting and do a research, here is the result.


## How Did JVM Detect Timezone {#how-did-jvm-detect-timezone}

All of the code can be found in this function: `private static synchronized TimeZone setDefaultZone()`

```java
  String zoneID = AccessController.doPrivileged(new GetPropertyAction("user.timezone"));

  // if the time zone ID is not set (yet), perform the
  // platform to Java time zone ID mapping.
  if (zoneID == null || zoneID.isEmpty()) {
      String javaHome = AccessController.doPrivileged(
              new GetPropertyAction("java.home"));
      try {
          zoneID = getSystemTimeZoneID(javaHome);
          if (zoneID == null) {
              zoneID = GMT_ID;
          }
      } catch (NullPointerException e) {
          zoneID = GMT_ID;
      }
}
```

First, it will check whether JVM has `user.timezone` property. If not, it will call this native method `getSystemTimeZoneID`, it was implemented in [java.base/share/native/libjava/TimeZone.c](https://github.com/AdoptOpenJDK/openjdk-jdk9u-backup-03-sep-2018/blob/master/jdk/src/java.base/share/native/libjava/TimeZone.c), and the main logic is in [java.base/unix/native/libjava/TimeZone_md.c](https://github.com/AdoptOpenJDK/openjdk-jdk9u-backup-03-sep-2018/blob/17007f6a09f553801fd424d3c71382717975f66d/jdk/src/java.base/unix/native/libjava/TimeZone_md.c).

In `Timezone_md.c`, it will find timezone by following steps, it will return the timezone immediately once found.

1.  Find `TZ` environment.
2.  Read `/etc/timezone`.
3.  Read `/etc/localtime`. If it is a soft link(ex: `/usr/share/zoneinfo/Asia/Shanghai`), return timezone by path. Otherwise, compare the content with all files in `/usr/share/zoneinfo`, if found, return timezone.
4.  Return `GMT` as timezone.


## How to Change Timezone {#how-to-change-timezone}

The available timezone in Linux can be listed by this command: `timedatectl list-timezones`


### Add JVM param {#add-jvm-param}

You can add `-Duser.timezone=Asia/Shanghai` as JVM parameters.


### Set TZ environment variable {#set-tz-environment-variable}

Add `export TZ=Asia/Shanghai` in `.bashrc`.


### Change `/etc/timezone` {#change-etc-timezone}

Set its content to `Asia/Shanghai`


### Change `/etc/localtime` {#change-etc-localtime}

Link it to `/usr/share/zoneinfo/Asia/Shanghai`


### Change timezone manually in Java Program {#change-timezone-manually-in-java-program}

All of these methods should work

-   Add this line before get time: `TimeZone.setDefault(TimeZone.getTimeZone("Asia/Shanghai"))`
-   Set JVM property by code `System.setProperty("user.timezone", "Asia/Shanghai")`
-   Set timezone manually in Calendar `Calendar.getInstance(TimeZone.getTimeZone("Asia/Shanghai"))`


## Ref {#ref}

1.  [How to Set the JVM Time Zone](https://www.baeldung.com/java-jvm-time-zone)
2.  [jvm linux 时区设置](https://cloud.tencent.com/developer/article/1175487)
3.  [Java default timezone detection, revisited](http://kaiwangchen.github.io/2018/09/30/java-timezone-revisited.html)
4.  [Java读取系统默认时区](https://www.cnblogs.com/darange/p/9368245.html)
5.  [How to set a JVM TimeZone Properly](https://stackoverflow.com/questions/2493749/how-to-set-a-jvm-timezone-properly/64415095#64415095)
