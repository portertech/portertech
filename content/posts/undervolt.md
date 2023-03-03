---
title: "Undervolt"
date: 2023-03-03
draft: false
toc: false
images:
tags:
  - linux
  - laptop
  - performance
  - tuning
  - battery
  - intel
  - undervolt
  - razer
---

I recently installed a fresh OS on my personal x86 laptop, the Razer
Stealth 13 (2020). One of the first things I like to do on a clean
Linux laptop is "undervolt" the CPU. Under-volting an Intel mobile CPU
will reduce CPU (and iGPU) temperatures while increasing battery
life - great right!? Certain CPUs can be under-volted more than
others, taking it too far will result in an unstable system (will
crash or be sluggish). My Razer Stealth has an i7-1065G7 which doesn't
like to be undervolted too much (but it does run at a 25W TDP!), you
will probably be able to be more aggressive.

### Installation

Install a little Python tool called "Undervolt".

```
sudo pip install undervolt
```

### Tuning

Begin with something modest. This example applies a -25mV undervolt to
the core and cache (generally need to be the same) and a -15mV
undervolt to the iGPU. This happens to be all I can do on my Razer
Stealth.

```
sudo undervolt --core -25 --cache -25 --gpu -15
```

### Stress Testing

I like to use two tools to test an under-volted CPU, "stress", and
"glmark2".

On Debian:

```
sudo apt install stress glmark2
```

Run stress with the appropriate number of workers for your CPU. I
typically use `max_threads - 1` to determine this value, which is 7
for the Razer Stealth's i7-1065G7 (4 cores, 8 threads).

```
stress --cpu 7
```

If the system remains stable for some time (e.g. >10 min), move on to
running glmark2 to load the iGPU.

```
glmark2
```

If everything remains stable, you weren't being aggressive enough :)
You can repeat the process until the system becomes unstable and then
dial it back.

### Persisting Changes

After finding the right values, create a systemd unit to apply them on
system boot.

```
sudo nano /etc/systemd/system/undervolt.service
```

/etc/systemd/system/undervolt.service (using my values)

```
[Unit]
Description=undervolt

[Service]
Type=oneshot
ExecStart=/usr/local/bin/undervolt -v --core -25 --cache -25 --gpu -15

[Install]
WantedBy=multi-user.target
```

Enable the unit.

```
sudo systemctl daemon-reload
sudo systemctl enable undervolt
```

Your system now runs cooler and has improved battery life!