# 🎮 homelab-retrogaming

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
                          └── TV box
```

The switch is plugged into the router, and the Sony Google TV and the TV box are plugged into the switch.

## Stack

- Linux Mint 22.3 Cinnamon
- Tailscale 1.104.1 for remote access
- Managed from the laptop over SSH (via Tailscale)
- Secure Boot enabled (Standard mode, HP PC, boot via `shimx64.efi`)
- NVIDIA driver 580.178.04 via DKMS, signed with a personal MOK key
- Steam + Proton for PC games
- Sunshine v2025.923.33222 (pinned) for streaming, web management on port 47990
- Moonlight on the Sony Google TV (wired Ethernet)

## Progress

| Step | Status |
|------|--------|
| Install Linux Mint 22.3 Cinnamon | ✅ Done |
| Install Tailscale | ✅ Done |
| SSH remote administration (key-based) | ✅ Done |
| NVIDIA 580 driver (DKMS, Secure Boot on, MOK-signed) | ✅ Done |
| Steam + Proton | ✅ Done |
| RetroDECK (NES to Wii U) | ⬜ Planned |
| Native source ports | ⬜ Planned |
| Sunshine + Moonlight streaming | ✅ Done |
| Controller-driven launcher menu | 💡 Idea |

## Log

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
