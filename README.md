# pulse-divider
*Simple pulse divider for irrigation flow meter*

[![Build](https://img.shields.io/travis/claybar/pulse-divider/master.svg)](https://travis-ci.org/claybar/pulse-divider)
[![Issues](https://img.shields.io/github/issues/claybar/pulse-divider/.svg)](https://github.com/claybar/pulse-divider/issues)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](https://opensource.org/licenses/MIT)

---
A simple pulse rate reducer to interface a high pulse-rate irrigation flow meter with a controller.  Developed for use with a 0-120 l/m Q4.5 water meter and an [OpenSprinkler irrigation controller](opensprinkler.com).

## Hardware

Developed and tested using a [DigiSpark](http://digistump.com/products/1) clone based on an ATTiny85.

The Opensprinler controller is primarilly designed for 2-wire dry-contact flow sensors but since the pulse IO is simply a GPIO input it can be driven with a 5V logic-level pulse train.  From the [2.1.7 user manual](https://openthings.freshdesk.com/support/solutions/articles/5000716364-user-manual-firmware-2-1-7-current-)
> Flow sensors with 3 wires and runs on 5V can also be used with OpenSprinkler. In this case,
insert the black wire (ground) to the right pin of the sensor terminal, data wire to the left pin of the sensor terminal, and
solder the red wire (5V) to the VIN pin on the circuit board.

## Interfacing

DigiSpark IO | Usage
------------ | ----------
P?           | Ground
P?           | +5V supply
P?           | Pulse in
P?           | Pulse out

## Maths

Calculating the reduction factor requries a bit of maths.  Starting with:

Pulse Meter
* 0-120 l/m
* Q = 4.5

Opensprinkler controller
* Maximum pulse rate 200 p/s ([see Rays comment](https://opensprinkler.com/forums/topic/3-wire-flow-sensor/#post-41339))
* Liters per pulse accuracy, 2 decimal places

At maximum flow rate the meter will produce a pulse frequency of `f = 120 * 4.5 = 540 Hz` which is in excess of the permitteed maximum of `200 p/s`.   The minimum pulse division is therefore `540/200 = 2.7`.  Non-integer division is more tricky to implement, so a candidate divisor of 10 would be suitable.  Checking this against the pulse measurement accuracy of the controller (2 dp) gives a volume per pulse of:
```
120 l / 60 s = 2 l/s
540 p/s / 27 = 20 p/s
2 l/s / 20 p/s = 0.1 l/p
```
This seems to fit well with th restrictions of OpenSprinkler, 10x under the maximum pulse rate and only requiring 1 decimal place of accuracy for the pulse volume.
