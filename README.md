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
> ### How to check kernel version on Linux?
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

> [!IMPORTANT]  
> ### Verifying the system’s response behavior.
> After rebooting, you should make sure that the new kernel is properly configured. This is indicated by whether the flags PREEMPT and RT are included in the output of the program uname.
> ```bash
>  uname -v | cut -d" " -f1-4 
> ```
> If the Output is something like `#1 SMP PREEMPT RT`, kernel is properly configured. 

<h2>Expected Time Span</h2>

The expected time span can be calculated with the rule of thumb

```
maximum_latency = clock_interval * 10⁵
```

This means that, for example, in a system with a clock frequency of 1 GHz and thus a clock interval of 1 ns, a maximum latency of less than 100 µs can be expected.

<h2>Plotting latency</h2>

- For measuring latency we are using `Cyclictest` from `rt-tests` package.
- It is one of the most frequently used tools for evaluating the relative performance of real-time systems.
- Cyclictest accurately and repeatedly measures the difference between a thread's intended wake-up time and the time at which it actually wakes up from clock_nanosleep in order to provide statistics about the system's latencies.
- It can measure latencies in real-time systems caused by the hardware, the firmware, and the operating system.

<h3>Installing rt-tests</h3>

Either clone the git repo and compile

```git
git clone git://git.kernel.org/pub/scm/utils/rt-tests/rt-tests.git
```
```bash
sudo apt-get install build-essential libnuma-dev
make
```
(or)

Install from apt in debian

```shell
sudo apt install rt-tests
```

> [!NOTE]
> We can use `cyclictest` command directly if installed via apt. We have to use `sudo ./cyclictest` from the appropriate directory if we are working with compiled src code.

<h3>Plotting Script</h3>

Run the [`plot.bash`](./plot.bash) Script file to produce `plot.png` and cpu core wise logs.

**Code**
```bash
#!/bin/bash

# 1. Run cyclictest
cyclictest -l100000 -m -Sp90 -i200 -h400 -q >output 

# 2. Get maximum latency
max=`grep "Max Latencies" output | tr " " "\n" | sort -n | tail -1 | sed s/^0*//`

# 3. Grep data lines, remove empty lines and create a common field separator
grep -v -e "^#" -e "^$" output | tr " " "\t" >histogram 

# 4. Set the number of cores, for example
cores=4

# 5. Create two-column data sets with latency classes and frequency values for each core, for example
for i in `seq 1 $cores`
do
  column=`expr $i + 1`
  cut -f1,$column histogram >histogram$i
done

# 6. Create plot command header
echo -n -e "set title \"Latency plot\"\n\
set terminal png\n\
set xlabel \"Latency (us), max $max us\"\n\
set logscale y\n\
set xrange [0:400]\n\
set yrange [0.8:*]\n\
set ylabel \"Number of latency samples\"\n\
set output \"plot.png\"\n\
plot " >plotcmd

# 7. Append plot command data references
for i in `seq 1 $cores`
do
  if test $i != 1
  then
    echo -n ", " >>plotcmd
  fi
  cpuno=`expr $i - 1`
  if test $cpuno -lt 10
  then
    title=" CPU$cpuno"
   else
    title="CPU$cpuno"
  fi
  echo -n "\"histogram$i\" using 1:2 title \"$title\" with histeps" >>plotcmd
done

# 8. Execute plot command
gnuplot -persist <plotcmd

```
