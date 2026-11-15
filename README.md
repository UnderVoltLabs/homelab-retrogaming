# 🎮 homelab-retrogaming

> [!IMPORTANT]
> **Final update (2026-11-15).** This is the last update of the homelab: the project is finished and
> **this repository will receive no further updates.**

A repurposed desktop being turned into a **solo gaming box** (work in progress): emulation from the
NES to the Wii U and PC games through Proton, all driven from a controller
and streamed to other screens in the house.

## Hardware

| Component | Model |
|-----------|-------|
| CPU | Intel Core i7-7700 (4 cores / 8 threads) |
| GPU | NVIDIA Quadro P4000 (8 GB VRAM, Pascal) |
| RAM | 16 GB |
| Storage | 2 TB HDD |
| Network switch | UGREEN 8-port 2.5G Ethernet switch (unmanaged, metal case, plug & play) |

The whole PC was bought for **$50**.

### Network wiring

```
Router ── UGREEN switch ──┬── Sony Google TV (wired Ethernet)
                          └── Gaming PC (the "box")
                                   ┆
                                   ┆  Tailscale (SSH), virtual link
                                   ┆
                              Laptop (anywhere)
```

## Stack

- Linux Mint 22.3 Cinnamon
- Tailscale 1.104.1 for remote access
- Managed from the laptop over SSH (via Tailscale)
- Secure Boot enabled (Standard mode, HP PC, boot via `shimx64.efi`)
- NVIDIA driver 580.178.04 via DKMS, signed with a personal MOK key
- Steam + Proton for PC games
- Sunshine v2025.923.33222 (pinned) for streaming, web management on port 47990
- Moonlight on the Sony Google TV (wired Ethernet), with **Frame pacing set to "Lowest latency"** (see [input lag](#input-lag-tips))
- Controller: 8BitDo Ultimate 2C on its **2.4 GHz USB dongle**, plugged directly into the USB port of the device that receives the controller (not Bluetooth)
- Monitoring on demand over SSH (`ssh administrator@retrogaming`): `btop` (CPU/RAM/disk/network) and `nvtop` (GPU), installed with `sudo apt install btop nvtop`. Nothing runs in the background.
- Custom GPU fan curve (`gpu-fan-curve` systemd service, Coolbits=4), see [gpu-fan/fan-curve.md](gpu-fan/fan-curve.md)
- Dummy plug forced to 1920x1080@60 before Sunshine (`set-1080p` user service), see [dummy-plug-1080p/dummy-plug-1080p.md](dummy-plug-1080p/dummy-plug-1080p.md)
- Firewall: `ufw` enabled (default deny incoming, allow outgoing), LAN `<LAN_SUBNET>` and `tailscale0` allowed, see [Security](#security)
- RetroDECK (Flatpak), added to Steam as a non-Steam game and launched from Big Picture with Steam Input

## Input lag tips

Playing through Moonlight on the Sony Google TV, input lag was too high with a Bluetooth controller. What fixed it:

1. **Controller on a 2.4 GHz USB dongle** instead of Bluetooth, plugged directly into the USB port of the device that receives the controller. Bluetooth (e.g. a Switch pad on the TV) works but has noticeably more lag.
2. **Moonlight → Settings → Frame pacing: "Lowest latency"** (instead of the smooth/balanced modes, which add buffering).

Also recommended:
- Enable the TV's **Game Mode** (disables image post-processing).
- Keep the TV on **wired Ethernet** (as now) or Wi-Fi 5 GHz. 2.4 GHz Wi-Fi shares the antenna and the band with Bluetooth and increases latency and interference.

## Security

Full hardening was **abandoned** on purpose, and only a firewall was set up:

- The box has no port open to the Internet. It is reachable only from the LAN and over Tailscale, and it is powered off about 95% of the time.
- - Firewall (`ufw`): default deny incoming, allow outgoing, then
  `sudo ufw allow from <LAN_SUBNET>` and `sudo ufw allow in on tailscale0`.
  Status confirmed: LAN `<LAN_SUBNET>` and `tailscale0` (v4 + v6) allowed. SSH/scp and Sunshine/Moonlight work from both the LAN and Tailscale.
- Rollback: `sudo ufw disable`.

## Progress

| Step | Status |
|------|--------|
| Install Linux Mint 22.3 Cinnamon | ✅ Done |
| Install Tailscale | ✅ Done |
| SSH remote administration (key-based) | ✅ Done |
| NVIDIA 580 driver (DKMS, Secure Boot on, MOK-signed) | ✅ Done |
| Steam + Proton | ✅ Done |
| RetroDECK (NES to Wii U): install, Steam Big Picture link, inputs | ✅ Done |
| Sunshine + Moonlight streaming | ✅ Done |
| Hardware monitoring (btop + nvtop, on demand over SSH) | ✅ Done |
| Custom GPU fan curve (systemd service) | ✅ Done |
| Controller input lag fix (2.4 GHz dongle + Moonlight frame pacing) | ✅ Done |
| Security hardening (full hardening abandoned, see [Security](#security)) | ❌ Abandoned |
| Firewall (`ufw`: LAN + Tailscale only) | ✅ Done |
| Resolution fix with a dummy DisplayPort plug (1080p60 user service) | ✅ Done |

## Log

### 2026-11-15 (final update)
- Firewall: the owner enabled `ufw` (default deny incoming, allow outgoing, `ufw allow from <LAN_SUBNET>`, `ufw allow in on tailscale0`). Status confirmed: LAN `<LAN_SUBNET>` and `tailscale0` (v4 + v6) allowed. SSH/scp and Sunshine/Moonlight work from the LAN and Tailscale. Rollback: `sudo ufw disable`.
- Decision: no full security hardening. No port is open to the Internet, the box is reachable only from the LAN and Tailscale, it is off about 95% of the time, and automatic updates could break the legacy NVIDIA 580 driver.
- **This is the final update of the homelab. The repository will receive no further updates.**

### 2026-11-15
- Problem: the dummy plug came up at 4096x2160 @ 29.97 Hz, so Sunshine streamed 4K at 30 Hz and Steam Big Picture looked tiny in Moonlight.
- Fix: `set-1080p.sh` forces the connected output (DP-7) to 1920x1080 @ 60 Hz with `xrandr`, run by the `set-1080p` systemd **user** service before `sunshine.service`.
- First attempt hooked the service to `xdg-desktop-autostart.target`, which Cinnamon never triggers, so the resolution was back to 4K after a reboot. Hooking it to `sunshine.service` fixed it. Confirmed after a real reboot.
- Sunshine: removed the `Low Res Desktop` app (it targeted a non-existent `HDMI-1`). Remaining: Desktop, Steam Big Picture, RetroDECK.
- Xorg `MetaModes` not used (a wrong value can cause a black screen). Procedure and files: [dummy-plug-1080p/dummy-plug-1080p.md](dummy-plug-1080p/dummy-plug-1080p.md).

### 2026-11-11
- Problem: too much input lag when playing through Moonlight on the Sony Google TV with a Bluetooth controller.
- Fix: new controller used on its **2.4 GHz USB dongle**, plugged directly into the USB port of the device that receives the controller, instead of Bluetooth; and Moonlight **Frame pacing set to "Lowest latency"**.
- The earlier Bluetooth setups (e.g. a Switch pad on the TV) are kept as fallbacks but have more lag.
- Tips: TV Game Mode on; TV on Ethernet or Wi-Fi 5 GHz (2.4 GHz Wi-Fi competes with Bluetooth). See [Input lag tips](#input-lag-tips).

### 2026-11-08
- Problem: with the stock fan curve, the GPU reached 80 °C after a 5-minute stress test.
- Fix: custom fan curve, a script run by a systemd service (`gpu-fan-curve`), with Coolbits=4 enabled in Xorg. If anything fails, the fan goes back to the driver's auto mode.
- Result: 62 °C on the same test. Procedure and scripts: [gpu-fan/fan-curve.md](gpu-fan/fan-curve.md).
- Hardware monitoring: installed `btop` and `nvtop` (`sudo apt install btop nvtop`), run from the laptop over SSH. Netdata removed (too heavy). Tip: two SSH terminals side by side while playing via Moonlight.
- Bought an 8BitDo Ultimate 2C controller (replaces the PowerA, plan changed from the Xbox controller).
- Known issue: with no screen plugged in, the PC has no display, so the resolution falls back to 1024x768 (4:3) and the Moonlight stream has black bars on the TV.
- Fix needed: buy a dummy plug (fake screen) for the GPU.
- Installed RetroDECK over SSH: `sudo flatpak install -y flathub net.retrodeck.retrodeck` (`sudo` required over SSH).
- First launch and configuration through Moonlight Desktop.
- Added RetroDECK to Steam as a non-Steam game.
- Inputs: enabled Steam Input in the RetroDECK properties; the controller shows up as an Xbox pad and the emulators are preconfigured.

### 2026-11-07
- Bug: after a reboot, Sunshine did not start and Moonlight could not see the PC.
- Fix 1: the Sunshine service did not start by itself, so added `~/.config/autostart/sunshine-start.desktop` ("Sunshine (start)" in Menu → Startup Applications).
- Fix 2: autologin was too slow (LightDM `autologin-user-timeout=1`), so set `autologin-user-timeout=0` in `/etc/lightdm/lightdm.conf`.
- Both are required. Confirmed after a real reboot.

### 2026-11-05
- Installed and configured Sunshine (v2025.923.33222). Pinned because the newer version (2026.914) does not support the GPU (NVENC fails on Pascal, LizardByte/Sunshine#5725). Web management on port 47990.
- Installed Moonlight on the Sony Google TV (wired Ethernet): 1080p60, 1 ms latency.
- Installed Steam and enabled Proton.
- Tested Dark Souls 1 streamed to the TV: runs fluidly at 60 FPS, 1080p, low latency.
- Known issue: the PowerA controller does not work well on PC. Plan: buy an Xbox controller.

### 2026-11-04
- Created my own signing key ("Homelab DKMS MOK") and enrolled it with `mokutil` (MokManager screen at reboot).
- Signed the NVIDIA 580.178.04 DKMS driver with that key (rebuilt with `dkms remove` + `dkms install`; `--force` does not re-sign).
- Re-enabled Secure Boot (Standard mode); `modinfo -F signer nvidia` shows the key and `nvidia-smi` works.

### 2026-11-03
- Installed Linux Mint 22.3 Cinnamon on the gaming box.
- Installed and connected Tailscale on the gaming box.
- Installed `openssh-server` and authorized the Windows laptop's ed25519 key.
- Added a `retrogaming` host alias in the laptop's SSH config; `ssh retrogaming` works.
- Installed the NVIDIA 580 driver, built with DKMS; `nvidia-smi` sees the Quadro P4000.
- Secure Boot had to be disabled in the HP BIOS for the driver to load.
