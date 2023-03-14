---
title: "GPU Max Blower"
date: 2023-03-10
draft: false
toc: false
images:
tags:
  - linux
  - server
  - performance
  - tuning
  - gpu
  - nvidia
  - sysadmin
  - devops
  - systemd
  - folding
  - pytorch
  - ml
---

I just want to share my new favourite systemd unit.

GPU MAX BLOWER! This unit sets the nvidia GPU fan speed to 100%, which
helps keep the GPU core and memory cooler under load. Most cards use a
"fan curve" to help reduce fan noise while keeping temperatures within
the specification. While this is acceptable (or preferred) in most
cases, a GPU may be running under load for extended periods, or in a
server enclosure where moving air quickly is critical. Maintain those
high clock speeds! This systemd unit only works if you have the nvidia
driver and associated tools installed.

`/etc/systemd/system/gpublower.service`

```
[Unit]
Description=GPU Max Blower

[Service]
Type=oneshot
ExecStart=/usr/bin/nvidia-settings -V -c :0 -a '[gpu:0]/GPUFanControlState=1' -a '[fan:0]/GPUTargetFanSpeed='"100"
RemainAfterExit=true
ExecStop=/usr/bin/nvidia-settings -V -c :0 -a '[fan:0]/GPUTargetFanSpeed='"80"
StandardOutput=journal

[Install]
WantedBy=multi-user.target
```

```
systemctl daemon-reload
systemctl start gpublower
systemctl status gpublower
```

You will see warnings in the log if the host doesn't use X11, this is
fine (it still works). The warnings will look like this.

```
Mar 10 01:17:14 node2 nvidia-settings[1585]: WARNING: NV-CONTROL Display not found.
Mar 10 01:17:14 node2 nvidia-settings[1585]: WARNING: NV-CONTROL Display not found.
Mar 10 01:17:14 node2 nvidia-settings[1585]: WARNING: Unable to determine number of NVIDIA 3D Vision Pro Transceivers on ':0'.
Mar 10 01:17:14 node2 nvidia-settings[1585]: WARNING: Unable to determine number of NVIDIA Display Devices on ':0'.
Mar 10 01:17:14 node2 nvidia-settings[1585]: WARNING: Unable to determine number of NVIDIA X Screens on ':0'.
Mar 10 01:17:14 node2 nvidia-settings[1585]:   Attribute 'GPUFanControlState' ([gpu:0]) assigned value 1.
Mar 10 01:17:14 node2 nvidia-settings[1585]:   Attribute 'GPUTargetFanSpeed' ([fan:0]) assigned value 100.
```

Check to see if your GPU fan is cranking. If there isn't a load on the
GPU, the service will have no effect.

```
watch nvidia-smi
```

If you have more than one nvidia GPU, you can append additional
arguments to the `ExecStart` command etc.

```
-a '[gpu:1]/GPUFanControlState=1' -a '[fan:1]/GPUTargetFanSpeed='"100"
```
