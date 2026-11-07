---
layout: post
title: "A Little Bit of Magic: Local Room Automation with Home Assistant and ESPHome"
date: 2026-11-07 12:45 +0300
categories: [Projects, Smart Home]
tags: [home-assistant, esphome, esp8266, docker, iot, automation]
media_subpath: /assets/img/Introductionha/
---

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Introduction](#introduction)
  - [Why Home Assistant](#why-home-assistant)
  - [Cost and technology groups](#cost-and-technology-groups)
  - [Defining the requirements](#defining-the-requirements)
  - [Selecting modules and sensors](#selecting-modules-and-sensors)
- [Power and wiring](#power-and-wiring)
  - [The 10 kΩ resistor for the light sensor](#the-10-kω-resistor-for-the-light-sensor)
- [System architecture](#system-architecture)
- [Deploying Home Assistant and ESPHome with Docker](#deploying-home-assistant-and-esphome-with-docker)
  - [Preparing the Linux host](#preparing-the-linux-host)
  - [The docker-compose.yml file](#the-docker-composeyml-file)
  - [How on\_multi\_click works](#how-on_multi_click-works)
  - [How the ESP controls the light without Home Assistant](#how-the-esp-controls-the-light-without-home-assistant)

---

## Introduction

Most of us believe in magic and would like controlling everyday things to be simple and effortless. It would be appealing to send our thoughts over a wireless network and be understood at the other end without saying a word. Can routine tasks be automated to make that vision at least partly attainable? This question led me to take a serious interest in home automation and smart-home solutions. The emergence of AI naturally invites dreams of having a capable butler, much like JARVIS, in one's own workshop. For now, however, I decided to start small: a minimal room-control system that operates locally and does not depend on the Internet.

### Why Home Assistant

While researching the market, I found a wide range of solutions. I will group them by installation effort, cost, and capabilities to provide a balanced comparison.

The first decision is which platform should serve as the system's central controller:

1. **[KNX.](https://www.knx.org/)** The benchmark for wired smart-home protocols. KNX systems offer exceptional fault tolerance, but they are difficult to configure and require cable routes to be planned during major renovation.
2. **[Wiren Board.](https://wirenboard.com/en/)** A popular industrial controller with an open Linux-based architecture for wired automation. It is somewhat easier to configure than KNX, but the system must likewise be designed and installed during renovation.
3. **[Home Assistant.](https://www.home-assistant.io/)** A powerful open-source platform for integrating a wide range of sensors, modules, and protocols. Its developer, Nabu Casa, offers ready-made hubs such as HA Green and HA Yellow, but the platform remains highly flexible: it can run on almost any hardware and be expanded with additional devices after renovation is complete.


### Cost and technology groups

* **KNX components.** They offer the highest fault tolerance, together with a premium price. They are an excellent choice for large properties designed to remain in service for decades.
* **Wiren Board.** Universal industrial controllers and expansion modules, including relays and RS-485/Modbus sensors. They can provide a reliable, premium-grade wired system at a substantially lower cost than KNX.
* **Zigbee / Matter devices.** Wireless solutions based on a mesh network topology. They offer considerable flexibility and good reliability at a fraction of the cost of industrial wired systems. Their main advantage is that they can be installed after renovation without disturbing the finishes.
* **[ESPHome.](https://esphome.io/)** A platform for customizing and building smart devices based on Espressif microcontrollers (ESP8266 / ESP32). It integrates seamlessly with Home Assistant. Boards and components are inexpensive, while a properly assembled and configured system can approach commercial solutions in reliability.


Based on these considerations, I chose **Home Assistant**.

The renovation was already complete and the electrical wiring was concealed behind the finishes. I therefore opted for wireless control nodes and micro-relays added to the existing switches.

Home Assistant can run on any inexpensive Linux machine. A used or new mini-PC, such as one based on the Intel N100, is often cheaper, faster, and more reliable than a Raspberry Pi, whose SD cards can be a point of failure.

Home Assistant has been an active open-source project for many years and has effectively become a standard for local smart-home installations. The platform continues to evolve: an increasing number of settings are moving from complex YAML files into a convenient web interface, making the system accessible not only to enthusiasts but also to ordinary users.

### Defining the requirements

The path to that kind of magic should be gradual. The first step is to determine how frequently used functions in a room can be automated. In my case, I chose lighting control; even this seemingly simple task allows for several interesting solutions.

The system should:

- **Control the main light** with a wall button. Not with an application or voice control, but with an ordinary button that is always at hand.
- **Turn the light on when motion is detected.** The light should turn on when someone enters and switch off after several minutes when the room is empty.
- **Measure room temperature and humidity** so that these values can later be used in automation scenarios, such as controlling the home's heating.
- **Measure the ambient light level.** This value will support the rule “do not turn the light on when the room is already bright enough.”
- **Control decorative RGB lighting.** This is primarily a matter of comfort, but it is what makes a room feel lived-in rather than like a laboratory.
- **Operate locally.** All of the above should continue to work when the external connection is unavailable.

These requirements determine the component set: one button, one motion sensor, one climate sensor, one photoresistor, one relay, one RGB strip, and one microcontroller.

### Selecting modules and sensors

The resulting list is as follows:

| Component | Purpose |
|---|---|
| ESP8266 (NodeMCU v2) | Wi-Fi microcontroller; an ESP32 is an option if expansion is planned |
| Momentary button | Manual control of the main light |
| PIR sensor (HC-SR505) | Motion detection |
| DHT11 | Temperature and humidity; the DHT22 is more accurate but more expensive |
| GL5528 photoresistor + 10 kΩ resistor | Voltage divider for estimating illuminance |
| Single-channel relay module | Switching the main light |
| RGB strip | Decorative lighting |
| Three-channel MOSFET module | PWM control of the strip |
| Mini560 PRO buck converter (5 V) | Power for the ESP8266, PIR sensor, and relay |
| Power supply for the strip | Selected according to the strip's voltage and current |
| Local Linux server | Home Assistant and ESPHome in Docker; an old laptop or mini-PC is sufficient |

Here is the reasoning behind each choice.

**ESP8266 versus ESP32.** I already had a NodeMCU v2, and it is sufficient for this project: it provides Wi-Fi and enough GPIO pins. It has only one analog input, but that is sufficient for the photoresistor. I would choose an ESP32 if I planned to add more sensors, Bluetooth, or a camera. For a minimal room system, the ESP8266 is a reasonable compromise between cost and simplicity.

![ESP32 development board.](esp32.jpg){: width="340" height="318" }
*Figure 1. ESP32 development board.*

**A momentary button rather than a latching switch.** A conventional switch has two states and two wires. A momentary button allows arbitrary logic to be assigned to different gestures: a short press can toggle the light, a long press can start a scenario, and a double press can switch everything off.
![Momentary wall button.](button.jpg){: width="320" height="320" }
*Figure 2. Momentary wall button.*


**PIR HC-SR505.** It is inexpensive and predictable, with sensitivity and delay fixed by the manufacturer. It operates at 5 V, and its output is compatible with the ESP8266. That is more than sufficient for a room. If adjustment is required, the HC-SR501, with its two trim potentiometers, is a better choice.

![HC-SR505 motion sensor.](move-sensor.jpg){: width="360" height="278" }
*Figure 3. HC-SR505 motion sensor.*


**DHT11.** Its accuracy is modest—±2 °C and ±5% relative humidity—but sufficient for household automation. For higher accuracy, use a DHT22 or a more modern sensor such as the SHT31.

![DHT22 temperature and humidity sensor.](dht22.jpg){:  width="320" height="320" }
*Figure 4. DHT22 temperature and humidity sensor.*


**A photoresistor with a 10 kΩ resistor.** This is a conventional voltage divider connected to input `A0`. The configuration's conversion factor to lux is only approximate and must be calibrated for the particular circuit; this is discussed separately in the wiring section.

![GL5528 photoresistor.](photorezistor.jpg){: width="360" height="294" }
*Figure 5. GL5528 photoresistor.*


**Relay module.** It switches the main light. Two details are important. First, relay modules may be active-HIGH or active-LOW, so the control polarity depends on the specific board. The configuration reflects this with the commented-out line `inverted: true`.

![Single-channel relay module.](relay.jpg){: width="360" height="291" }
*Figure 6. Single-channel relay module.*


**MOSFET module for RGB lighting.** The microcontroller's GPIO pins are not designed to carry the strip's current. The strip therefore has a separate power supply, while the microcontroller only controls the MOSFET gates. The configuration uses three PWM channels. Check the channel order on the particular module and strip in practice, since the colors may be swapped.

![Three-channel MOSFET module.](mostfet-board.jpg){: width="360" height="270" }
*Figure 7. Three-channel MOSFET module.*


**Local server.** There is no need to purchase dedicated hardware. An old laptop or a Linux mini-PC is an ideal candidate. Home Assistant and ESPHome run in Docker, so the entire system can reside on a single device placed unobtrusively in a corner.

![Linux mini-PC used as a local server.](mini-pc.jpg){: width="320" height="320" }
*Figure 8. Linux mini-PC used as a local server.*

---

## Power and wiring

The circuit is divided into four power domains. This separation isolates the power stage, logic, and sensors and helps prevent voltage drops on sensitive components.

![ESP8266 wiring schematic for room automation.](esp8266-smart-room-schematic.jpg)
*Figure 9. ESP8266 wiring schematic for room automation.*

**Circuit 1—12 V primary supply.** The LED strip is powered by the common 12 V supply. A MOSFET module switches the R, G, and B channels in response to signals from the ESP controller.
![12 V power supply for the LED strip.](power.avif)
*Figure 10. 12 V power supply for the LED strip.*

**Circuit 2—12 V to 5 V conversion.** A buck DC-DC converter supplies 5 V from the same 12 V source. This rail powers the ESP controller, relay module, and motion sensor.

**Circuit 3—3.3 V sensor supply.** The photoresistor and the DHT11 temperature-and-humidity sensor are connected to this rail. The DHT11 must be powered at **3.3 V**: its output signal can reach the supply voltage, and at 5 V the signal level would exceed what the ESP8266 can safely accept. Supplying it at 3.3 V eliminates the need for level shifting.

**Circuit 4—220 V mains load.** The relay switches the main light. Its control input `IN` must recognize a **3.3 V** logic level; some conventional 5 V modules may not trigger reliably at 3.3 V. When working with 220 V mains, an enclosure, proper insulation, and compliance with electrical-safety requirements are essential.

The **HC-SR505 motion sensor** remains on the 5 V rail because its output signal is 3.3 V and is fully compatible with the ESP8266 without a level shifter.

### The 10 kΩ resistor for the light sensor

On its own, a photoresistor does not produce a voltage; it changes resistance according to the ambient light level. To allow the ESP8266 to read this change through analog input `A0`, the photoresistor is connected as a **voltage divider** with a fixed **10 kΩ** resistor.

The wiring is as follows:

- Connect one lead of the photoresistor to **3.3 V**.
- Connect the other lead to **A0** and to one lead of the 10 kΩ resistor.
- Connect the other lead of the 10 kΩ resistor to **GND**.

The divider's midpoint, between the photoresistor and the 10 kΩ resistor, is connected to `A0`. In bright conditions, the photoresistor's resistance decreases and the voltage at `A0` rises. In darkness, its resistance increases and the voltage at `A0` falls. The 10 kΩ resistor forms the lower leg of the divider and determines its measurement range: a value that is too high reduces sensitivity in darkness, while a value that is too low drives the divider toward saturation in bright conditions. A 10 kΩ resistor is a standard compromise for the GL5528 and most household applications.

> **Important:** boards such as the NodeMCU and Wemos D1 Mini already include a divider on input `A0`, so applying up to 3.3 V is safe. Bare ESP8266 modules, such as the ESP-12, accept only **0–1.0 V** on their analog input. The same 3.3 V divider must not be connected directly to them; an additional divider or an external ADC such as the ADS1115 is required.
{: .prompt-danger }

---

## System architecture

Before configuring the system, it is useful to examine how its hardware and software interact. The architecture consists of four layers: hardware, firmware, a local server, and an interface.

**Hardware.** The controller and its connected sensors. The controller polls the sensors, operates the relay and LED strip, and sends data to the server over Wi-Fi.

**Firmware.** ESPHome builds firmware from the YAML configuration and uploads it to the microcontroller—first over USB and subsequently via OTA. The firmware establishes the Wi-Fi connection, API, and OTA update service, and also contains the local logic: the button switches the relay directly without waiting for a server response.

![ESPHome device page for the Bedroom Main controller](assembled-controller.jpg)
*Figure 11. ESPHome device page for the Bedroom Main controller, showing its controls, sensor states, related automation, and motion activity history.*

**Local server.** Home Assistant and ESPHome run in Docker on one Linux machine. ESPHome builds and flashes the devices, while Home Assistant handles automation, the user interface, and history. The server is not required for basic operation: even when it is off, the button and sensors on the ESP8266 continue to follow their local rules.

**Interface.** The Home Assistant web dashboard and mobile application. This is where sensor states are displayed and scenarios are configured, but the button signal does not pass through it on its way to the relay.

![Home Assistant Bedroom dashboard showing light controls and sensor readings](home-assistant-dashboard.jpg)
*Figure 12. Home Assistant Bedroom dashboard with controls for the main and RGB lights and readings from the temperature, humidity, illuminance, and motion sensors. The interface shown here may change in future releases.*

Docker was chosen not as a matter of fashion, but for reproducibility. The `ghcr.io/home-assistant/home-assistant` and `ghcr.io/esphome/esphome` images are official and update predictably; reverting to an earlier version requires changing a single line in `docker-compose.yml`. All state—configuration, database, and firmware—is stored in two host volumes, so the system can be moved to another machine by copying the directories.

---

## Deploying Home Assistant and ESPHome with Docker

### Preparing the Linux host

The server requirements are minimal:

- Linux with [Docker](https://docs.docker.com/engine/install/) and the Compose plugin (`docker compose`) installed.
- A user in the `docker` group, so commands do not have to be run through `sudo`.
- A user in the `dialout` group (on Ubuntu/Debian); otherwise the ESPHome container cannot access `/dev/ttyUSB0` to flash the board over USB.
- Ports `8123` (Home Assistant) and `6052` (ESPHome) available.

To add the current user to the required groups, run the following commands and restart the session (or reboot the server):

```bash
sudo usermod -aG docker \$USER
sudo usermod -aG dialout \$USER
```

### The docker-compose.yml file

The `docker-compose.yml` file from my project is as follows:

```yaml
services:
  homeassistant:
    container_name: homeassistant
    image: ghcr.io/home-assistant/home-assistant:stable
    privileged: true
    restart: unless-stopped
    environment:
      - TZ=Europe/Moscow
    volumes:
      - /home/roboeggs/homeassistant:/config
      - /run/dbus:/run/dbus:ro
    network_mode: host

  esphome:
    container_name: esphome
    image: ghcr.io/esphome/esphome
    privileged: true
    restart: unless-stopped
    environment:
      - TZ=Europe/Moscow
      - ESPHOME_USERNAME=${ESPHOME_USERNAME}
      - ESPHOME_PASSWORD=${ESPHOME_PASSWORD}
    devices:
      - /dev/ttyUSB0:/dev/ttyUSB0
    volumes:
      - /home/roboeggs/homeassistant/esphome:/config
      - /etc/localtime:/etc/localtime:ro
    network_mode: host
```

Before starting the stack, create a `secrets.yaml` file in the same directory as `docker-compose.yml` and add:

```yaml
ESPHOME_USERNAME=your_login
ESPHOME_PASSWORD=Your_password
```
These credentials are required to sign in to the ESPHome web interface.

After Docker Compose starts, Home Assistant will be available at `http://<IP-address>:8123`, and ESPHome at `http://<IP-address>:6052`. The web interfaces allow devices to be created and flashed and automations to be configured without manually editing YAML files.

First, create a Home Assistant account. Then open ESPHome, create a device, select the board type (NodeMCU v2), and assign a name.

The YAML editor will then open, allowing you to define the device configuration. If the pins have not been changed and the wiring follows the diagram above, the ready-made configuration below can be used. It includes all settings required for the button, motion sensor, photoresistor, DHT11, relay, and RGB strip. It is not necessary to paste the entire YAML file. Instead, consider each block, which is separated by comments, and add the blocks individually. The `api` and `wifi` sections will be device-specific, but ESPHome populates them automatically when the device is created.

![ESPHome Device Builder dashboard showing the online Bedroom Main device](esphome-device-list.jpg)
*Figure 13. ESPHome Device Builder dashboard showing the online Bedroom Main device, its YAML file, and the installed ESPHome and Device Builder versions.*

```yaml
esphome:
  name: bedroom-main
  friendly_name: "Bedroom Main"
  area: "Bedroom"

esp8266:
  board: nodemcuv2

logger:
  level: WARN

api:
  encryption:
    key: "******************************************"

ota:
  - platform: esphome

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  ap:
    ssid: "esp8266******************"
    password: "**************"

captive_portal:

binary_sensor:
  - platform: gpio
    pin:
      number: GPIO0
      mode: INPUT_PULLUP
      inverted: true
    name: "Wall Button Raw"
    internal: true
    filters:
      - delayed_on_off: 50ms
        
    on_multi_click:
      - timing:
          - ON for at most 1s
          - OFF for at least 0.3s
        then:
          - light.toggle: light_bulb
  
      - timing:
          - ON for at most 1s
          - OFF for at most 1s
          - ON for at most 1s
          - OFF for at least 0.2s
        then:
          - light.toggle: rgb_led

  - platform: gpio
    pin: GPIO5
    device_class: motion
    name: "Room Motion Sensor"

output:
  - platform: esp8266_pwm
    pin: GPIO14
    id: pwm_blue
  - platform: esp8266_pwm
    pin: GPIO12
    id: pwm_red
  - platform: esp8266_pwm
    pin: GPIO13
    id: pwm_green

  - platform: gpio
    pin: GPIO4
    id: light_relay_output

light:
  - platform: binary
    name: "Main Light"
    id: light_bulb
    output: light_relay_output

  - platform: rgb
    name: "RGB Light"
    id: rgb_led
    red: pwm_red
    green: pwm_green
    blue: pwm_blue
    effects:
      - random:
      - strobe:
      - flicker:
      - pulse:

sensor:
  - platform: dht
    pin: GPIO2
    model: DHT11
    temperature:
      name: "Room Temperature"
    humidity:
      name: "Room Humidity"
    update_interval: 1min

  - platform: adc
    pin: A0
    name: "Light Level"
    id: light_level
    device_class: illuminance
    state_class: measurement
    update_interval: 1min
    unit_of_measurement: '%'
    filters:
      - multiply: 100.0
```

After the configuration is saved, ESPHome will validate its syntax and offer to flash the device. The initial flash is performed over USB; subsequent updates can be delivered over Wi-Fi using OTA. Connect the board to a computer and select the firmware through the ESPHome interface. The Docker configuration exposes the USB device so that the firmware can be uploaded to the board. Once flashing succeeds, the device will appear in Home Assistant, where the light and RGB strip can be controlled and sensor data viewed through the web interface.

### How on_multi_click works

The `on_multi_click` block is a state machine that tracks how long the button remains `ON` (pressed) and `OFF` (released). ESPHome reads these timings and compares them with the defined patterns.

The logic can be understood step by step:

1. **Single click**
   
```yaml
timing:
  - ON for at most 1s
  - OFF for at least 0.3s
then:
  - light.toggle: light_bulb
```

* `ON for at most 1s`: The controller waits for a press. If the button is held for longer than one second, this pattern is cancelled, leaving the possibility of assigning a different function to a long press.

* `OFF for at least 0.3s`: This is the key step. After the button is released, ESPHome starts a 0.3-second timer. If the button is not pressed again during that interval, the system identifies the event as a single click and executes `light.toggle: light_bulb`. These 0.3 seconds are the source of the apparent delay before the light turns on.
  
2. **Double click**
   
```yaml
timing:
  - ON for at most 1s
  - OFF for at most 1s
  - ON for at most 1s
  - OFF for at least 0.2s
then:
  - light.toggle: rgb_led
```

* `ON for at most 1s` -> `OFF for at most 1s`: You press and release the button. The system detects the release and waits for no more than one second. If you press it again during that interval, it proceeds to the next step. If you do not, the single-click rule from step 1 is triggered.

* `ON for at most 1s`: You press the button a second time. If it is held for no longer than one second, the pattern continues.

* `OFF for at least 0.2s`: You release the button for the second time and do not press it again for 0.2 seconds. The double click is confirmed and `light.toggle: rgb_led` is executed.

ESPHome evaluates the rules from top to bottom. If the input does not match the first pattern because the second press occurs before 0.3 seconds have elapsed, it proceeds to the second pattern and recognizes the double click.

### How the ESP controls the light without Home Assistant

This is one of ESPHome's main advantages. All logic written in the YAML file is compiled into binary code and uploaded directly to the ESP8266 microcontroller's memory.

The process of switching on the light is as follows:

    You physically press the button.

    The voltage on GPIO0 changes.

    The ESP8266 firmware—not the server, but the chip itself—immediately recognizes the press through the on_multi_click block.

    The microcontroller sends a signal to GPIO4, which is connected to the relay.

    The relay clicks and the light turns on.

Where is Home Assistant at this point? Nowhere. It is not involved in this process at all.

Home Assistant simply “listens” to the ESP8266 over the local network through the API. When the ESP switches the relay, it sends Home Assistant a notification: “I have just switched on Main Light.” Home Assistant updates the dashboard so that the light's current state is visible.

If the server is shut down, the Internet cable is unplugged, or the Wi-Fi router is turned off, the button will still switch the light on and off. The controller does not need a network for this; it only needs power. In this arrangement, Home Assistant serves as a convenient control panel and statistics system, not as a required intermediary.
