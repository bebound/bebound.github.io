# Control QNAP NAS Fan Speed in PVE


It's getting hot in summer, the hard disks in my QNAP NAS are overheating. Since I'm using PVE, I can't control the fan speed. Luckily, somebody has already solved this problem. Here is the guide.


## Install the 8528 Kernel Module {#install-the-8528-kernel-module}

First, install the 8528 kernel module to get the fan speed information. In Linux, you can install the `lm-sensors` package to get the CPU and HDD temperature. You can run `sensors` command to see the temperature. However, the fan speed information is not shown. As the QNAP uses a ITE8528 embedded controller to control the fan speed, the `lm-sensors` package does not support it.


### Option 1: Install with DKMS {#option-1-install-with-dkms}

[qnap8528](https://github.com/0xgiddi/qnap8528) provides a kernel module to support this controller. It contains the instruction to install with DKMS and [autoload module on startup with systemd](https://github.com/0xgiddi/qnap8528#autoload-module-on-startup-with-systemd). Don't forget to install the pre-requisite packages: `apt install build-essential pve-headers dkms`. You may need to install `proxmox-headers-$(uname -r)` if the kernel is not the latest version.

This methods automatically rebuild the kernel module when the kernel is updated.


### Option 2: Build module in Docker {#option-2-build-module-in-docker}

Gzxiexl created [a more conventient script](https://github.com/gzxiexl/qnap8528) to build, install and automatically load the kernel module. This script does not support PVE 9 currently, but I've created a [PR](https://github.com/gzxiexl/qnap8528/pull/1). Before it's merged, you need to install the kernel headers with `apt install proxmox-headers-$(uname -r)` and edit the `Dockerfile` to use `debian:13` as base image. Then you can run the `sudo build.sh`.

This methods is more convenient, but you need to rebuild the kernel module when the kernel is updated.

After install the kernel module, the fan speed and NAS temperature appear in the `sensors` output.

```nil
root@pve:~# sensors
qnap8528-isa-0000
Adapter: ISA adapter
fan1:         834 RPM
temp1:        +42.0°C
temp6:        +41.0°C
pwm1:             62%

drivetemp-scsi-1-0
Adapter: SCSI adapter
temp1:        +44.0°C  (low  =  -5.0°C, high = +80.0°C)
                       (crit low = -10.0°C, crit = +85.0°C)
                       (lowest = +38.0°C, highest = +54.0°C)
```

The above output contains the `drivetemp-scsi-1-0`, which shows the hard disk temperature. It's not shown by default, you need to load the `drivetemp` module with `sudo modprobe drivetemp`


## Cooler Control {#cooler-control}

Second, [Cooler Control](https://docs.coolercontrol.org/installation/debian.html) can control the fan speed with customized fan speed curve, which is based on the CPU or HDD temperature. After install the debian package, don't forget to edit the `/etc/coolercontrol/config.toml` to change the bind IP addresses. The service can be started with  `systemctl enable --now coolercontrold`, and the web UI is available at `http://<your-pve-host>:11987`. There is a pretty dashboard to show CPU and HDD temperature in the web UI. By setting a fan speed curve profile, the fan speed can be controlled with the HDD temperature.

{{< figure src="/images/CoolerControl.png" >}}

Now the hard disks can be kept in a safe temperature range.


---

> Author: KK  
> URL: https://fromkk.com/posts/control-qnap-nas-fan-speed-in-pve/  

