---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "NVIDIA JetPack Flashing Issues Q&A"
subtitle: ""
summary: "JetPack 4.2 Flashing Issues and how to resolve"
authors:
- admin
tags:
- Jetson
categories: ["Misc"]
date: 2019-11-06T22:52:31-05:00
lastmod: 2019-11-06T22:52:31-05:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: true

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---





## Jetson usb-device mode issues

####  *Q1: How to verify if usb-device mode is working?*



A: After flashing and Jetson device booting up, usb-device mode is up . The expected results at this stage are:

- 192.168.55.1 is assigned to Jetson device, and

- 192.168.55.100 is assigned to host machine.



#### *Q2: usb-device mode is already up and running, however, 192.168.55.100 not assigned on host. Why?*

A: One possible reason is that usb-ethernet network interface is disabled on host machine? To enable it, 

1. Disconnect and Connect the USB cable connecting the jetson and Host-PC, and check the dmesg log.
   You might see a log similar to this:
   [107833.388835] cdc_ether 2-2:1.5 usb1: register 'cdc_ether' at usb-0000:00:1 4.0-2, CDC Ethernet Device, 72:40:f7:72:c9:fa
2. Next go to Settings->Network and check all the wired connections for the same MAC-address.
3. In the top right corner there will be an option to enable it.
4. Next follow the procedure as provided by the SDK manager to install the SDK components



#### *Q3: I tried thousands of ways, but the SDK Manager still reports "could not detect nvidia jetson device connected to usb", what can I do?*



A: You can also use wired cable to connect your host machine and Jetson board, and then install everything. Just make sure they are assigned to appropriate IP address:

- 192.168.55.10 (not 1) is assigned to Jetson device, and

- 192.168.55.100 is assigned to host machine.



## References

1. [NVIDIA Forums - JetPack 4.2 Flashing Issues and how to resolve](https://devtalk.nvidia.com/default/topic/1050477/jetson-tx2/jetpack-4-2-flashing-issues-and-how-to-resolve/)