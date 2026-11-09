# GPU fan curve

A custom fan curve for the Quadro P4000 in the gaming box. The stock curve is tuned for silence and lets the card run hot under load; this one spins the fan up earlier.

| | Peak GPU temperature (5-minute stress test) |
|---|---|
| Stock curve | 80 °C |
| Custom curve | 62 °C |

## The curve

| GPU temperature | Fan speed |
|---|---|
| 40 °C or less | 40 % |
| 50 °C | 50 % |
| 60 °C | 65 % |
| 70 °C | 85 % |
| 75 °C or more | 100 % |

Speeds between two points are interpolated linearly. The fan speeds up immediately but slows down by at most 2 % every 2 seconds, so it does not oscillate.

## How it works

| File | Installed to | Purpose |
|---|---|---|
| [`gpu-fan-curve.sh`](gpu-fan-curve.sh) | `/usr/local/bin/` | Reads the GPU temperature every 2 s and sets the fan speed with `nvidia-settings` |
| [`gpu-fan-curve.service`](gpu-fan-curve.service) | `/etc/systemd/system/` | Starts the script after LightDM and restarts it if it dies |
| `20-nvidia-coolbits.conf` | `/etc/X11/xorg.conf.d/` | Enables `Coolbits=4`, which allows manual fan control |

`nvidia-settings` needs a running X server, so the script uses LightDM's display (`DISPLAY=:0`, `XAUTHORITY=/var/run/lightdm/root/:0`). This relies on the autologin setup from 2026-11-07.

### Fail-safe

- `Coolbits=4` only unlocks fan control: no overclocking, no voltage changes.
- If the temperature can't be read or `nvidia-settings` fails, the fan goes back to the driver's automatic curve.
- When the script stops for any reason (stop, reboot, crash, `kill -9`), the service's `ExecStopPost` hands the fan back to the driver, then restarts the script after 5 s.
- The card still has its own hardware protection (throttling, then shutdown). See the thresholds with `nvidia-smi -q -d TEMPERATURE`.

## Install

### 1. Enable Coolbits (once)

```bash
sudo mkdir -p /etc/X11/xorg.conf.d
sudo tee /etc/X11/xorg.conf.d/20-nvidia-coolbits.conf >/dev/null <<'EOF'
Section "Device"
    Identifier "NVIDIA GPU"
    Driver "nvidia"
    Option "Coolbits" "4"
EndSection
EOF
sudo reboot
```

After the reboot, `grep -i coolbits /var/log/Xorg.0.log` should show `"Coolbits" "4"`.

### 2. Install the script and the service

Copy the two files to the box (for example with `scp gpu-fan-curve.* retrogaming:/tmp/` from the laptop), then on the box:

```bash
sudo install -m 755 /tmp/gpu-fan-curve.sh /usr/local/bin/
sudo install -m 644 /tmp/gpu-fan-curve.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now gpu-fan-curve
```

## Verify

```bash
journalctl -u gpu-fan-curve -n 5 --no-pager   # expect "Fan curve active: ..."
```

Then watch the temperature and fan speed in `nvtop` while running [gpu-burn](https://github.com/wilicc/gpu-burn) in a second SSH session: `./gpu_burn 300`.

## Change the curve

Edit the `CURVE=` line in `/usr/local/bin/gpu-fan-curve.sh` (`temperature:speed` pairs), then `sudo systemctl restart gpu-fan-curve`.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Log shows `nvidia-settings failed (X not ready or Coolbits missing)` | X is not running, or Coolbits was not applied | Check `pgrep -a Xorg` and `grep -i coolbits /var/log/Xorg.0.log`. The fan stays on auto meanwhile. |
| Log shows `Cannot read GPU temperature` | `nvidia-smi` is not responding | Check `nvidia-smi` and the driver. The fan stays on auto meanwhile. |
| Black screen after enabling Coolbits | X rejected the config | Over SSH: `sudo rm /etc/X11/xorg.conf.d/20-nvidia-coolbits.conf && sudo reboot` |

## Uninstall

```bash
sudo systemctl disable --now gpu-fan-curve
sudo rm /etc/systemd/system/gpu-fan-curve.service /usr/local/bin/gpu-fan-curve.sh
sudo systemctl daemon-reload
sudo rm /etc/X11/xorg.conf.d/20-nvidia-coolbits.conf && sudo reboot   # optional
```
