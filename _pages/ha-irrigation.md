---
title: "Home Assistant & Home brewed Autonomous irrigation"
permalink: /ha/irrigation
date: 2023-04-29T18:38:52+00:00
layout: single
toc: true
toc_label: "ToC"
toc_sticky: true
---

# Design

## Hardware
1. Water tank: it obviously provides with the irrigation main item: water. Yoohoo ! :+1:.
2. Solar panel: it provides electrical power to activate the pump.
3. Pump: needed to push water to the plant, through pipe and drippers.
4. Switch: provides external input to the pump.

## Software
* At some point, software is needed to drive the switch to start the pump, and stop the pump too.
* I happen to have a [Home Assistant](http://home-assistant.io) instance running in the apartment, connected to wifi.
* Home assistant is provided with tons of marvellous addons. This [addon](https://github.com/jeroenterheerdt/HAsmartirrigation) from @jeroenterheerdt is able, given meteorological data, to calculate evapotranspiration and water volume to give plants for them to get enough water. This means, that depending on the temperatures, the rain of the day, wind, ... the netto evapotranspiration is assessed and the water quantities to provide are calculated. This is nice piece of software, and the developer is really keen to help (I even made some Pull Requests to adapt it a bit to my needs)

## Big picture

The global picture is the following:

![big picture]({{ "/images/index/schema.drawio.png" | relative_url }}){: .align-center}

## Hardware Details

### Actuator

This is piece of hardware which actuates the pump to do its job, following the orders from Home Assistant brain.
It is based on a Sonoff 4CHPROR3.

![Sonoff 4CHPROR3]({{ "/images/index/sonoff.jpg" | relative_url }}).

Following [this tutorial](https://flobul-domotique.fr/flasher-le-sonoff-4ch-pro-r3/) I was able to flash [ESPHome](https://esphome.io) which is a (again!) wonderful piece of software, owned by Home Assistant developpers (hence the "Home" is ESPHome), which is so easy to interface with Home Assistant you wouldn't believe. ESPhome has support for [Sonoff 4CH](https://esphome.io/devices/sonoff_4ch.html) which is compatible with Sonoff 4CHProR3. It allowed me to wipeout the firmware with which is is shipped, to my own custom one.

### Pump

The pump is a 12V DC powered pump, used for swimming pool, installed inside the water tank, powered by the battery through the actuator.

### Battery

![Battery]({{ "/images/index/battery.jpg" | relative_url }})

The battery is a motorcycle one, 12V DC, charged by the solar panel through the regulator.

### Solar charge controller

The regulator is actually the solar charge controller: it charges the battery with solar power, and powers the Actuator through its 12V DC output.

![Solar charge controller]({{ "/images/index/regulator.jpg" | relative_url }})

### Solar panel

Nothing fancy there, I chose a 50W 12V one, place it on the roof of a cabinet I have on the terrace, and connected it to the solar charge controller.

## Software

### Home Assistant

Like I said I happen to have a Home Assistant instance running on my network. Just added ESPHome and configured it .

### ESPhome

Actuator is configured with the following YAML description :

```yaml
esphome:
  name: switch-pompe
  platform: ESP8266
  board: esp01_1m

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

switch:
  - platform: gpio
    name: "Sonoff 4CH Relay 1"
    pin: GPIO12
    id: pompe
```

The switch interface can be used from automations or scripts through the

```yaml
- service: switch.turn_on
  target:
    entity_id: switch.sonoff_4ch_relay_1
  data: {}
```

# FAQ
1. I noticed the title, what do you mean by "Autonomous" ?
	* I happen to live in a apartment with a terrace in town. This terrace has no easy water access, and no electrical powering facility. 
2. What does this mean for an irrigation system ?
	* This means:
		* Being able to pump water to plants, without grid electricity
		* Being able to store resources: water, power
		* Be scarce on resources: just use what is needed, no waste (if possible)

