# CUSTOM-PL1-PL2-LIMITS-ARCH-LINUX
This is a custom setup I have made that can automatically change PL1/PL2 limits on your CPU by utilizing PPD power profiles, which is a great power saving option, aswell as an amazing way to lower temps on laptops.
Tested on an Alienware M17 R4 Featuring an i7-10870h, 32GB DDR4, RTX 3080 16GB, CachyOS Linux with KDE.

# Intel PL Limits by Power Profile

## Create a Script

```bash
sudo nano /usr/local/bin/set-pl-by-profile.sh
```

## Use this as a template

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

Create:

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
