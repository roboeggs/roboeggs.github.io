---
layout: post
title: "Smart Light: An Embedded, Privacy-Preserving Video Surveillance Station"
date: 2020-07-19 12:00 +0000
categories: [Projects]
tags: [embedded-to-systems, 3d-models]
media_subpath: /assets/img/crimecamera/
---

In 2018, I participated with a team in Secur'IT Cup, a hackathon organized by Kaspersky Lab in Budapest. At the time, devices such as the Nvidia Jetson or Intel Movidius had not yet become the ubiquitous, off-the-shelf accelerators for on-device neural network inference that they are today; running deep learning models on embedded hardware was still very much an emerging practice rather than a standard engineering choice. Our objective was to determine whether such constrained hardware could support a complete next-generation video surveillance system. The resulting prototype was recognized as one of the winning submissions.

## Problem Statement

By 2018, advances in deep learning and computer vision had already begun to reshape video surveillance across retail, banking, and public safety: in-store promotional analytics, biometric authorization in financial institutions, and automated crime detection in public spaces. However, existing commercial solutions presented customers with two unsatisfactory options:

- **On-premises deployment**, in which the client bears full responsibility for infrastructure maintenance and software compatibility — a substantial operational burden.
- **Cloud-based analytics**, which is often costly and, more critically, insecure. Biometric data is frequently stored and processed in the cloud without encryption, and even where standard encryption is applied, the data must be decrypted at the point of computation — reintroducing the very vulnerability the encryption was meant to eliminate.

Our approach addressed both limitations simultaneously: an inexpensive hardware station that performs computationally intensive inference locally, combined with homomorphic encryption, a cryptographic technique that permits mathematical operations to be performed directly on ciphertext. This allows the server to compare and match facial data without ever decrypting it.

## System Architecture

Each camera station, built around a Raspberry Pi equipped with an Intel Movidius neural compute stick, performs face detection and recognition locally, on-device. The resulting facial feature vectors (descriptors) are homomorphically encrypted before transmission to the cloud, where they are stored as ciphertext — meaningless to any observer, including the service provider itself.

The retrieval workflow proceeds as follows: an operator seeking to locate footage of a specific individual submits a reference photograph. This query image is transformed into an encrypted descriptor and compared against the encrypted descriptors stored in the cloud database using a purpose-built homomorphic similarity function. Matching results are returned to the client in encrypted form and are decrypted only on the client side, meaning the server never has access to plaintext facial data at any stage of the pipeline.

In terms of performance, the on-device pipeline achieved face detection and recognition at approximately 6 FPS, while similarity search against a database of one million faces required roughly 20 milliseconds per query. Because the primary computational load is distributed across the camera stations rather than concentrated on a central server, the architecture scales efficiently with the number of deployed units.

## My Contribution: Hardware and Mechanical Engineering

My role on the project concerned the electrical and mechanical engineering of the camera station. The device, referred to as the "Smart Light" station, was designed to mount on street poles and integrates:

- a camera mounted on a custom two-degree-of-freedom (2-DOF) moving platform, fabricated via 3D printing;
- an LED floodlight for scene illumination;
- a four-microphone array for acoustic source localization;
- a Raspberry Pi paired with an Intel Movidius neural compute stick for on-device inference.

![Exploded diagram of the Smart Light station's 3D-printed body, showing the outside and inside views](light-station-diagram.webp)

![3D-printed enclosure with the camera mounting ring and internal brackets installed](enclosure-camera-mount.jpg)

I modeled the enclosure and all mechanical components in KOMPAS-3D and produced them using 3D printing.

![Early 3D-printed enclosure halves, prior to mounting internal components](enclosure-halves-early.jpg)

A further part of my work involved integrating these components — servo actuators, environmental sensors (temperature, humidity, pressure), the camera module, and the compute unit — into a single coherent embedded system within a compact, weatherproof enclosure suitable for outdoor deployment.

![Inside view of two assembled stations, showing the Raspberry Pi boards, Movidius compute sticks, and wiring](internals-wiring.jpg)

## Related Work

At the time, several companies — NtechLab, Vocord, and VisionLabs — already offered face recognition solutions, and some provided proprietary video management systems. However, none combined proprietary camera hardware, large-scale face search capability, and secure (encrypted) data processing within a single offering. Our system was designed to unify these three properties.

## Outcome

![Two assembled camera stations mounted on test stands during indoor demonstration](stations-on-stands.jpg)
 
![The completed Smart Light unit mounted overhead, with the LED floodlight, static light, and sensor board visible](final-mounted-unit.jpg)

Video demonstration of the prototype's operation:
{% include embed/youtube.html id='ygWDG8WTIJs' %}

The resulting product concept, the Crime Camera Station, was structured around a straightforward business model: clients would purchase the camera hardware and subscribe to a REST API providing video analytics services. Prospective pilot partners discussed during the project included Ak Bars Bank, the Moscow Department of Information Technologies, and an industrial chemical manufacturer.

This project required integrating an unusually broad technical stack within a short timeframe — spanning embedded hardware design, mechanical fabrication, deep learning, and applied cryptography. What I recall most vividly is the constant shift between hands-on hardware work and CAD modeling, and the satisfaction of seeing all of these disparate components converge into a functioning prototype.

The full pitch deck presented at Secur'IT Cup 2018 is available [here](assets/pdf/crimecamera/CrimeCamera.pdf).
