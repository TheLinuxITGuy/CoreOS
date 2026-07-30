# Fedora CoreOS Butane Configs

Butane configs for provisioning Fedora CoreOS (FCOS) via Ignition.

📺 **YouTube walkthrough:** [Add link here]

## 1. AdGuard Home (Quadlet)

Deploys [AdGuard Home](https://adguard.com/en/adguard-home/overview.html) as a rootless Podman Quadlet container on FCOS.

- Creates `core` user with SSH key auth
- Creates data dirs: `/var/adguardhome/work` and `/var/adguardhome/conf`
- Disables `systemd-resolved`'s stub listener (frees port 53 for AdGuard)
- Defines a Quadlet unit (`adguardhome.container`) running `adguard/adguardhome:latest` in host network mode, with the data dirs mounted in
- Repoints `/etc/resolv.conf` to the real upstream DNS resolver instead of the stub

**File:** `AdGuardHomeQuadlet.bu`

## 2. Base FCOS Config

Minimal starting config for a Fedora CoreOS instance.

- Creates `core` user with SSH key auth

**File:** `Basic.bu`

## Usage

Convert each Butane config to an Ignition file before provisioning. 
> [!NOTE]
> Replace `coreos.bu` with the file you are actually using.

```bash
podman run -i --rm quay.io/coreos/butane:release --strict < coreos.bu > coreos.ign
```
