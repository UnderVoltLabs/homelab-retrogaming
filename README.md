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

## Stack

- Linux Mint 22.3 Cinnamon
- Tailscale 1.104.1 for remote access
- OpenSSH server, key-based, from the Windows admin laptop

## Progress

| Step | Status |
|------|--------|
| Install Linux Mint 22.3 Cinnamon | ✅ Done |
| Install Tailscale | ✅ Done |
| SSH remote administration (key-based) | ✅ Done |
| NVIDIA 580 driver | ✅ Done |
| Steam + Proton | ⬜ Planned |
| RetroDECK (NES to Wii U) | ⬜ Planned |
| Native source ports | ⬜ Planned |
| Sunshine + Moonlight streaming | ⬜ Planned |
| Controller-driven launcher menu | 💡 Idea |

## Log

### 2026-11-03
- Installed Linux Mint 22.3 Cinnamon on the gaming box.
- Installed and connected Tailscale on the gaming box.
- Installed `openssh-server` and authorized the Windows laptop's ed25519 key.
- Added a `retrogaming` host alias in the laptop's SSH config; `ssh retrogaming` works.
- Installed the NVIDIA 580 driver, built with DKMS; `nvidia-smi` sees the Quadro P4000.
- Secure Boot had to be disabled in the HP BIOS for the driver to load.
