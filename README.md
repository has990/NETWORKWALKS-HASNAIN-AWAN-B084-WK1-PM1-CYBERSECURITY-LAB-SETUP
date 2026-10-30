# VMware-Based Cybersecurity Lab Setup — WK1-PM1

<p align="center">
  <strong>Networkwalks Cybersecurity Program · Batch B084</strong><br>
  <em>Kali Linux virtual lab using VMware Workstation</em>
</p>

## Project overview

This repository documents the implementation of the WK1-PM1 cybersecurity testing lab using **VMware Workstation** and **Kali Linux**. The supplied module names VirtualBox as its reference hypervisor; VMware was used here because the Kali VM was already installed and operational in VMware. The required lab networking and integration outcomes were retained.

## Objectives

- Create a private NAT-backed lab network using `10.0.0.0/24`.
- Configure Kali Linux with static address `10.0.0.2/24`.
- Provide internet connectivity through NAT and verify gateway/DNS access.
- Share the Windows host Downloads folder with Kali.
- Enable VMware clipboard copy/paste and drag-and-drop settings.
- Create and verify a VMware snapshot.

## Lab architecture

| Component | Configuration |
|---|---|
| Hypervisor | VMware Workstation |
| Virtual network | VMnet8 (NAT) |
| Subnet | `10.0.0.0/24` |
| NAT gateway | `10.0.0.1` |
| Kali interface | `eth0` |
| Kali IPv4 | `10.0.0.2/24` (static) |
| DHCP range | `10.0.0.128–10.0.0.254` |
| DNS | `1.1.1.1`, `8.8.8.8` |
| Shared folder | Windows Downloads → VMware share `Downloads` → Kali `/mnt/hgfs/Downloads` |
| Snapshot | `Kali-Linux-Lab-Configured` |

## Configuration summary

### 1. VMware network

VMnet8 was configured as NAT on subnet `10.0.0.0/24`, with NAT gateway `10.0.0.1`. The DHCP pool was moved to `10.0.0.128–10.0.0.254`, keeping the statically assigned Kali address outside the dynamic range.

### 2. Kali static IP

NetworkManager profile: `Wired connection 1`.

```bash
sudo nmcli connection modify "Wired connection 1" ipv4.addresses 10.0.0.2/24
sudo nmcli connection modify "Wired connection 1" ipv4.gateway 10.0.0.1
sudo nmcli connection modify "Wired connection 1" ipv4.dns "1.1.1.1 8.8.8.8"
sudo nmcli connection modify "Wired connection 1" ipv4.method manual
sudo nmcli connection down "Wired connection 1" && sudo nmcli connection up "Wired connection 1"
```

### 3. Connectivity verification

The supplied terminal evidence shows:
- Gateway `10.0.0.1`: 4 packets received, 0% loss.
- DNS lookup and ping to `google.com`: successful resolution and 4 packets received, 0% loss.
- Final interface and route: `eth0 = 10.0.0.2/24`, default route via `10.0.0.1`.

### 4. Shared folder

VMware Shared Folders was set to **Always enabled**. The host Downloads directory was shared with the guest as `Downloads`. Kali discovered it with `vmware-hgfsclient`, then mounted it:

```bash
sudo mkdir -p /mnt/hgfs
sudo vmhgfs-fuse .host:/ /mnt/hgfs -o allow_other
ls /mnt/hgfs/Downloads
```

A FUSE compatibility warning appeared, but the captured test shows the share mounted and its contents accessible. If it is not mounted after a reboot, run the mount command again.

### 5. Guest integration and snapshot

VMware Guest Isolation has both **Enable copy and paste** and **Enable drag and drop** checked. These settings were enabled; a separate end-to-end transfer test was not captured.

Snapshot Manager confirms the snapshot `Kali-Linux-Lab-Configured` exists. A snapshot is a restore point, not a standalone backup.

## Troubleshooting notes

**Issue: DHCP range invalid after changing VMnet8 subnet.**  
Cause: VMware retained the old `192.168.16.x` DHCP range while the subnet had changed to `10.0.0.0/24`.  
Resolution: Changed the range to `10.0.0.128–10.0.0.254`, matching the new subnet.

**Issue: Shared folder was detected but `/mnt/hgfs` did not exist.**  
Cause: The shared folder was visible to VMware Tools, but the guest mount point had not been created/mounted.  
Resolution: Created `/mnt/hgfs`, mounted with `vmhgfs-fuse`, and verified `/mnt/hgfs/Downloads`.

## Evidence

Screenshots captured during the actual VMware/Kali setup are stored in [evidence/](evidence/). The shared-folder terminal screenshot has been redacted to conceal unrelated host filenames while retaining evidence of successful mounting and access.

| Evidence | Screenshot |
|---|---|
| VMware VM network adapter | [01-vm-network-adapter.png](evidence/01-vm-network-adapter.png) |
| VMnet8 NAT subnet | [02-vmnet8-nat-network.png](evidence/02-vmnet8-nat-network.png) |
| DHCP pool | [03-dhcp-pool.png](evidence/03-dhcp-pool.png) |
| Kali static IP and route | [04-kali-static-ip-route.png](evidence/04-kali-static-ip-route.png) |
| Connectivity test | [05-connectivity-tests.png](evidence/05-connectivity-tests.png) |
| Shared folder settings | [06-shared-folder-settings.png](evidence/06-shared-folder-settings.png) |
| Shared folder access (redacted) | [07-shared-folder-access-redacted.png](evidence/07-shared-folder-access-redacted.png) |
| Clipboard and drag/drop | [08-guest-isolation.png](evidence/08-guest-isolation.png) |
| Snapshot creation | [09-snapshot-creation.png](evidence/09-snapshot-creation.png) |
| Snapshot Manager confirmation | [10-snapshot-manager.png](evidence/10-snapshot-manager.png) |

## Report

- [PDF project report](docs/VMware_Cybersecurity_Lab_Setup_Report_Hasnain_Awan.pdf)
- [Editable Word report](docs/VMware_Cybersecurity_Lab_Setup_Report_Hasnain_Awan.docx)

## Scope and safety

This environment is intended for authorised cybersecurity learning and testing in a controlled lab. The source module's optional Phase 2 (additional Windows/Android VMs) was not included in this setup. The platform substitution from VirtualBox to VMware Workstation is disclosed rather than represented as a VirtualBox implementation.

---
**Prepared by:** Hasnain Awan · **Batch:** B084 · **Task:** WK1-PM1
