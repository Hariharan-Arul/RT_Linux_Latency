<h1 align="center"> A Case Study On Latency in Real Time Linux </h1>
In order to fulfill the requirements of a real-time system, a system must react to an external event like an interrupt within a defined time frame.

<h2>Steps to give a Linux kernel, Real Time properties</h2>
<ul>
  <li>download the kernel</li>
  <li>download the PREEMPT_RT patch</li>
  <li>patch the kernel</li>
  <li>build the kernel</li>
  <li>restart your system</li>
  <li>choose the RT kernel</li>
</ul>

<h3>Download the kernel</h3>

First select the appropriate RT kernel, whose version is as close as possible to the kernel version of the respective distribution.

> [!TIP]
> How to check kernel version on Linux?
> ```bash
> uname -r
> ```
> or
> ```bash
> cat /proc/version
> ```

Suppose we get <br>
`Linux version 5.15.0-122-generic` <br>
This means you have kernel major version 5, minor version 15, patch 0 and sublevel 122

<h3>Download the PREEMPT_RT patch</h3>

Get the kernel that is closest to the current non-RT version from [Linux_RT](https://cdn.kernel.org/pub/linux/kernel/projects/rt/) page. <br>
`patch-5.15.167-rt79.patch.xz` is the closest from the page.

<h3>Patch the kernel</h3>

With the following commands the appropriate source code is downloaded, the patch is downloaded, the archive is unpacked and the patch applied:

```bash
mkdir -p /usr/src/kernels
cd /usr/src/kernels
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.15.122.tar.gz
wget https://cdn.kernel.org/pub/linux/kernel/projects/rt/5.15/patch-5.15.167-rt79.patch.xz
tar xf linux-5.15.122.tar.gz
mv linux-5.15.122 linux-5.15.122-rt79
cd linux-5.15.122-rt79
xz -d ../patch-5.15.167-rt79.patch.xz
patch -p1 <../patch-5.15.167-rt79.patch
```

To ensure that the RT kernel supports the current distribution as well as possible, the first step is to take over its kernel configuration, which is located in the /bootdirectory of Debian, as with many other distributions, and which matches the kernel version number. 

<br>

In this case, it is the file `/boot/config-5.15.0-122-generic`, which is now copied to the root directory of the kernel source code under the name `.config`:

```bash
cp /boot/config-5.15.0-122-generic .config
```

<h3>Build the kernel</h3>

In the last step, before the kernel can be compiled, the new kernel has to be configured so that the functionality imported with the RT patch is also used. The command make menuconfig is called and we select Processor type and features -> Preemption Model -> Fully Preemptible Kernel (RT).

The kernel is then compiled, installed, and the system must be rebooted.

```bash
make -j4 
make modules_install install
```

<h3>Restart your system and choose the RT Kernel</h3>

```bash
sudo reboot
```



