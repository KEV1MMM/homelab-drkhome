# DRKHOME — Personal Cybersecurity Homelab

A home lab built for hands-on blue team / SOC practice, running on repurposed hardware. This project documents the setup, configuration, and real troubleshooting encountered while building a virtualization environment for security training (Active Directory attacks/detection, SIEM practice, network segmentation).

## Purpose

Practical platforms like TryHackMe and HackTheBox provide attacker-side infrastructure, but don't let you build and defend your own environment end-to-end. This homelab fills that gap — a place to simulate real corporate scenarios (AD, detection tooling, firewalls) and generate genuine incident write-ups for my portfolio.


## Software Stack

- **Hypervisor:** Proxmox VE 9.2
- **Base OS:** Debian (Trixie)
- **Planned VMs:** Windows Server (Active Directory), Kali Linux, Security Onion, Splunk ...

## Setup Summary

1. Wiped Windows, installed Proxmox VE via USB installer (balenaEtcher)
2. Configured static network on the local subnet
3. Verified Intel VT-x/VT-d virtualization support in BIOS
4. Resolved network connectivity (see [troubleshooting.md](./troubleshooting.md))
5. Post-install hardening: disabled enterprise repo, cleaned old kernel, set CPU governor to performance

## Access

Managed entirely headless via the Proxmox web UI (`https://<host-ip>:8006`) from a separate machine — no direct interaction with the physical hardware after initial install.

## Troubleshooting Log

Real issues encountered during setup, documented as incident reports: [troubleshooting.md](./troubleshooting.md)

## Roadmap

- [ x ] First VM: Ubuntu Server (baseline test)
- [ ] Kali Linux VM
- [ ] Windows Server + Active Directory
- [ ] Security Onion (detection/SIEM)
- [ ] Splunk
- [ ] pfSense (isolated internal network segmentation practice)
