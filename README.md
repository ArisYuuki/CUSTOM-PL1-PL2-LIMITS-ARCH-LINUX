# CUSTOM-PL1-PL2-LIMITS-ARCH-LINUX
![Arch Linux](https://img.shields.io/badge/Arch-Linux-blue?logo=arch-linux)
![CachyOS](https://img.shields.io/badge/CachyOS-Optimized-brightgreen)
![Intel](https://img.shields.io/badge/CPU-Intel-blue)

Fine‑tune Intel CPU power limits on Arch Linux laptops.

This repository provides a simple systemd + script solution to automatically apply custom PL1 / PL2 power limits based on your active power profile. It integrates with Linux’s native powerprofilesctl interface so you get consistent performance tuning, better thermals, and optimized power efficiency on Intel‑based systems.

# Intel PL Limits by Power Profile

## Create a Script

```bash
sudo nano /usr/local/bin/set-pl-by-profile.sh
```

## Use this as a template
PL1 is your long power limits. This will be used to determine what baseline you want your CPU to run at while under load. a good baseline is to set your laptops PL1 to 10w LESS then the TDP it is rated for. In this case, my i7-10870h runs at 45w without turbo, but I actually have deminishing returns at about 38w in terms of temperature vs performance. setting it to 35w in balanced mode can lower my temperatures by 10C under load without my fans going crazy.

PL2 is the short power limits. This is used to determine the power limits for when the CPU is using the turbo to keep the system running stable during rapid changes. This essentially gives the CPU the ability to process moments in games where it goes from calm to chaos (think in the event of a first person shooter when you're running to the battlefield, to when you get into a massive gunfight with explosions and lots of things on screen moving) without having a massive dip in your framerate.

TAU for each of these will determine how long the CPU will boost for or hold that sustained clock speed for. Generally, longer PL1/PL2 times result in higher sustained clock speeds, but also more generated heat. having the PL2 Tau set between 2-8 seconds can keep your performance snappy, without melting the system.

```bash
#!/bin/bash

RAPL="/sys/class/powercap/intel-rapl:0"

PROFILE=$(powerprofilesctl get)

set_limits() {
  echo "$1" > $RAPL/constraint_0_power_limit_uw
  echo "$2" > $RAPL/constraint_1_power_limit_uw
  echo "$3" > $RAPL/constraint_0_time_window_us
  echo "$4" > $RAPL/constraint_1_time_window_us
}

case "$PROFILE" in
  performance)
    # PL1 70W, PL2 110W, Tau 56s / 8s
    set_limits 70000000 110000000 56000000 8000000
    ;;
  balanced)
    # PL1 35W, PL2 60W, Tau 20s / 3s
    set_limits 35000000 60000000 20000000 3000000
    ;;
  power-saver)
    # PL1 28W, PL2 30W, Tau 16s / 2s
    set_limits 28000000 30000000 16000000 2000000
    ;;
esac
```

## Make it Executable by using the following command

```bash
sudo chmod +x /usr/local/bin/set-pl-by-profile.sh
```

## Create a Systemd Service

```bash
sudo nano /etc/systemd/system/pl-profile.service
```

Paste:

```ini
[Unit]
Description=Set Intel PL limits based on power profile
After=multi-user.target power-profiles-daemon.service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/set-pl-by-profile.sh

[Install]
WantedBy=multi-user.target
```

## Trigger on Profile Change so you can change PL1/PL2 Limits based on power profile
Doing this will tell your system to adjust the PL1/PL2 Limits every time you use PPD and change between Power-Save/Balanced/Performance modes. This is useful if your laptop (like mine) has fan curves tied directly to PPD. in my case, when using the Balanced Power Profile on, my laptop fans will only ramp up to the same speeds they would in AWCC (alienware command center for windows) under the performance mode which is 3900rpm. and when switching to the Performance Power Profile, it would allow fans to ramp up to 5400rpm, alowing for more thermal headroom.

```bash
sudo nano /etc/systemd/system/pl-profile.path
```

Paste:

```ini
[Unit]
Description=Watch for power profile changes

[Path]
PathChanged=/var/lib/power-profiles-daemon/state.ini

[Install]
WantedBy=multi-user.target
```

## Enable Everything

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now pl-profile.service
sudo systemctl enable --now pl-profile.path
```
