![picokit-10-button-debounce](https://raw.githubusercontent.com/mytechnotalent/picokit-10-button-debounce/main/picokit-10-button-debounce.png)

<br>

## FREE Reverse Engineering Self-Study Course [HERE](https://github.com/mytechnotalent/reverse-engineering)
## FREE Embedded Hacking Course [HERE](https://github.com/mytechnotalent/Embedded-Hacking)

<br>

# PICOKIT-10 BUTTON DEBOUNCE

### Debounced Button Edges and Authenticated Heartbeat
#### Lesson 10 of the Picokit Series

<br>

***
**LEGAL DISCLAIMER:**
The information, tools, and code provided in this repository and course are strictly for educational, research, and defensive purposes only.

You are explicitly prohibited from using any materials contained herein to access, test, modify, or exploit any device, network, or system that you do not own 100% or for which you do not have explicit, documented, and legally binding authorization to interact with.

By using this repository and course, you acknowledge and agree that:

1. Any illegal, unauthorized, or malicious use of this information is solely your responsibility.
2. The author(s) and contributor(s) of this repository and course shall not be held liable for any damages, legal repercussions, criminal charges, or unauthorized actions resulting from the use, misuse, or abuse of the contents herein.
3. You will comply with all applicable local, state, national, and international laws regarding cybersecurity and computer fraud.

**IF YOU DO NOT AGREE WITH THESE TERMS, DO NOT USE THIS REPOSITORY AND COURSE.**
***

<br>
<br>

## Overview

The tenth Picokit lesson. The node reads a tactile button on GP15
and accepts exactly one press edge per thirty millisecond
debounce window, counting the accepted presses on the red, yellow,
and green lamps, and every five seconds it transmits an
authenticated heartbeat over LoRa to a Python gateway that logs
and displays it. It reuses the standard node shape and adds edge
detection and contact-bounce suppression.

<br>

## What it teaches

- Configuring a tactile switch as an input with the internal pull-up.
- Accepting exactly one edge per contact debounce window.
- Counting edges and mapping the count onto several GPIO outputs.
- Sealing a tiny JSON body with Argon2id and XChaCha20-Poly1305 and
  sending it with an AT+SEND over the RYLR998.
- The gateway side: receive, authenticate, reject, log, and display.

<br>

## Hardware

| Peripheral | Pico 2 pin | Role |
| --- | --- | --- |
| Tactile button | GP15 | debounced edge input |
| Red / Yellow / Green | GP16 / GP18 / GP17 | press count |
| Onboard LED | GP25 | heartbeat, one blink per tick |
| RYLR998 | GP8 TX / GP9 RX | LoRa heartbeat |
| Debug Probe | SWCLK/SWDIO/GND, GP0/GP1 | SWD and the console |

<br>

## How it works

The node runs `monitor_step` in a loop. Every 200 ms it offers the
button to the debouncer, which returns true at most once per thirty
millisecond window, increments a press counter, and shows the count
on the three lamps, blinking GP25 once per tick, and every
5 seconds it seals `{"n":10,"s":<seq>,"k":<presses>}` with the
field key and sends it over LoRa. The gateway authenticates each
frame and only then parses it.

<br>

## Build and flash

```bash
cd firmware
cmake -S . -B build -G Ninja -DPICO_BOARD=pico2 -DPICO_PLATFORM=rp2350-arm-s
cmake --build build
openocd -f interface/cmsis-dap.cfg -f target/rp2350.cfg \
  -c "program build/picokit_10_button_debounce.elf verify reset exit"
```

<br>

## Watch the node

Open the console at 115200 and reset:

```text
BOOT
DEBOUNCE k=0 seq=0
DEBOUNCE k=1 seq=0
DEBOUNCE k=2 seq=0
RX from 0x0001, N bytes
```

<br>

## The gateway

```bash
cd gateway
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python3 listen.py --port /dev/cu.usbserial-A50285BI --hub 0001 --network 18 --db gateway.db
```

It prints `OK node=10 rssi=...` per authenticated heartbeat.
The terminal dashboard `python3 tui.py --db gateway.db` and the web dashboard
`python3 web/app.py --db gateway.db` show the same rows.

<br>

## Verify

```bash
python3 .opencode/skill/embedded-c-standard/audit_c_standard.py
python3 .opencode/skill/embedded-python-standard/audit_python_standard.py
python3 .opencode/skill/iot-readme-standard/validate_readme.py
python3 .opencode/skill/iot-banner-standard/validate_banner.py
python3 scripts/run_tests.py
python3 scripts/check_coverage.py
```

<br>

# Next
[picokit-11-button-events](https://github.com/mytechnotalent/picokit-11-button-events)

<br>

# License
[MIT License](https://github.com/mytechnotalent/picokit-10-button-debounce/blob/main/LICENSE)
