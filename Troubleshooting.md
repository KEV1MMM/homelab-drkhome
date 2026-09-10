# Troubleshooting Log

Real issues encountered while setting up DRKHOME, documented as incident reports. Kept in SOC-style format (symptom → diagnosis → root cause → resolution) as practice for real-world documentation.

---

## Incident 001 — No network connectivity after install

**Symptom:**
After completing the Proxmox installation, the web UI (`##############`) was unreachable from another machine on the same network. Browser returned `ERR_CONNECTION_REFUSED`.

**Diagnosis:**
ip a
showed the WiFi interface (`wlp0s20f3`) in `state DOWN`, while the bridge (`vmbr0`) was configured to use it as its bridge-port.

**Root Cause:**
The Proxmox installer never prompted for WiFi credentials during setup. The bridge was configured against a wireless interface with no active association to the network — it had an IP "on paper" but no real link. Additionally, WiFi is generally unreliable for Proxmox bridging, since bridged networking depends on multiple MAC addresses passing through a single interface, which most WiFi adapters/routers don't handle well.

**Resolution:**
Purchased a USB-C to Gigabit Ethernet adapter. Connected it, brought the interface up manually:

ip link set enx9c6###### up
Edited `/etc/network/interfaces`, changing `bridge-ports nic0` (the pinned WiFi interface) to `bridge-ports enx9c6#######`. Restarted networking:

systemctl restart networking
Confirmed `vmbr0` came up with `state UP` and `LOWER_UP`, and the web UI became reachable.

**Lesson:** For any Proxmox install intended to run bridged VM networking, plan for wired Ethernet from the start — don't rely on WiFi even for initial setup.

---

## Incident 002 — DNS resolution failing (scripts/updates unable to reach the internet)

**Symptom:**
Running `curl` against a GitHub raw URL failed with `Could not resolve host`. `nslookup github.com` returned `communications error to 1###.###.###: timed out`.

**Diagnosis:**
- `ping -c 4 8.8.8.8` succeeded (with some packet loss), confirming raw internet connectivity was present.
- `ping -c 4 192.168.0.1` (the router) failed entirely — later determined to be expected, since many consumer routers block ICMP to themselves while still routing traffic and DNS normally.
- `/etc/resolv.conf` appeared correctly configured with `nameserver 1###.####.####`, later `8.8.8.8` / `1.1.1.1` — but `nslookup` kept querying `1##.###.###` regardless of edits.

**Root Cause:**
A file-naming typo. Edits were repeatedly being made to `/etc/resolve.conf` (extra "e") instead of the actual `/etc/resolv.conf`, creating a second, unused file. The real config file was never touched by the "fixes," which is why DNS kept failing against the router's DNS service despite the router itself being reachable for general traffic.

**Resolution:**
Identified the duplicate file:

find /etc/resolve.conf
rm /etc/resolve.conf
Correctly edited `/etc/resolv.conf` with:
nameserver 8.8.8.8
nameserver 1.1.1.1
Confirmed resolution:

nslookup github.com
→ Address: 140.82.113.4

**Lesson:** Always double-check exact file paths before assuming a persistent config issue — a single misplaced character can look identical to a "settings not saving" bug.

---

## Incident 003 — `apt update` failing with 401 Unauthorized

**Symptom:**
After running the community post-install script (`community-scripts/ProxmoxVE`), `apt update` failed:

- Err:6 https://enterprise.proxmox.com/debian/pve trixie InRelease
401 Unauthorized

  **Diagnosis:**
Proxmox ships with an "enterprise" repository enabled by default, intended for users with a paid support subscription. This install uses the free/community edition, with no valid subscription credentials — so any request to the enterprise repo is rejected.

**Root Cause:**
The `pve-enterprise` repository remained active alongside the free `pve-no-subscription` repo the post-install script added. `apt update` attempted to query both, and failed on the enterprise one.

**Resolution:**
Commented out the enterprise repo:

nano /etc/apt/sources.list.d/pve-enterprise.list

deb https://enterprise.proxmox.com/debian/pve trixie pve-enterprise

Re-ran `apt update` — completed successfully against the free repositories only.

**Lesson:** Community post-install scripts don't always fully disable enterprise repos even when adding the free ones — worth manually verifying `sources.list.d/` after running any automated setup script.

---

## Incident 004 — Proxmox web UI unreachable after system updates

**Symptom:**
After completing `apt update`/system changes, the web UI became unreachable again (`ERR_CONNECTION_REFUSED`), despite networking (Incident 001) having already been fixed.

**Diagnosis:**

systemctl status pveproxy

showed the service was not running.

**Root Cause:**
The `pveproxy` service (which serves the Proxmox web interface) had stopped, likely due to a restart triggered during package updates.

**Resolution:**

systemctl restart pveproxy
systemctl enable pveproxy

Confirmed `active (running)` and re-verified web UI access.

**Lesson:** After any system-level update on a headless server, explicitly verify core services are still running rather than assuming a restart handled it cleanly.






























