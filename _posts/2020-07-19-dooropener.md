---
layout: post
title: Smart Office Door Opener with FaceID Access
date: 2020-07-19 12:00 +0000
categories: [Projects]
tags: [embedded-to-systems, 3d-models]
media_subpath: /assets/img/dooropener/
---
![The door release button](door-button.jpg)

Controlling access to an office door can be simple—until you’re not allowed to modify the door itself. In this project, the challenge was exactly that: only authorized employees should be able to open the door, but any physical changes to the door mechanism were strictly prohibited.

To solve this, I designed a compact mechanical device that physically presses the standard door‑opening button. Instead of wiring into the door system, the device interacts with it just like a human finger would.

![Front view. Screenshot from the CAD system.](screenshot-of-the-3d-model.jpg)

![Rear view. Screenshot from the CAD system.](screenshot-of-the-3d-model-2.jpg)

![Side view. Screenshot from the CAD system.](screenshot-of-the-3d-model-3.jpg)

## How the Access System Worked

The door was opened only after successful FaceID authentication. Once the system verified the user, it sent a command over the local Wi‑Fi network to trigger the device. This allowed secure, contactless access without touching the door hardware.


## Hardware and Design

The core of the device was built around a WEMOS D1 mini microcontroller, connected to a servo motor that performed the actual button press. The housing and mounting parts were 3D‑printed, making it easy to adapt the design to the office environment and the exact placement of the button.

The device was installed next to the door panel and worked fully autonomously. After receiving the Wi‑Fi command, it activated the servo, pressed the button, and granted access.

![Front view. The assembled device.](assembled-device.jpg)

![Rear view. The assembled device.](assembled-device-1.jpg)

## Demonstration Video

Below is a short demonstration of how the system worked in practice.

{% include embed/youtube.html id='PAPhdNjdJwc' %}

## Conclusion

This project was done quite a while ago, but it still shows a practical way to solve a restriction without modifying the original door hardware. The device presses the button after receiving a Wi‑Fi command triggered by FaceID verification. A simple, functional setup that met the requirements and worked reliably.
