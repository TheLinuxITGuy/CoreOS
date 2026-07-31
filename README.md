# Fedora CoreOS Butane Configs

Butane configs for provisioning Fedora CoreOS (FCOS) via Ignition.

## 🎬 Video

[![Video](https://img.youtube.com/vi/57y_n-X93_4/maxresdefault.jpg)](https://youtu.be/57y_n-X93_4)

## Available Configs

Grab the `.bu` file for the container you need:

* **AdGuardHomeQuadlet.bu**: Deploys AdGuard Home as a rootless Podman Quadlet container, handles data directories, frees port 53 by disabling the systemd-resolved stub listener, and runs on the host network.
* **Basic.bu**: Minimal starting config that creates the `core` user with SSH key authentication.
 > [!NOTE]
> I'll be adding to this list over time.

## Usage

Convert your chosen Butane config to an Ignition file using Podman. The `.ign` file will be needed by the CoreOS installer.

```bash
podman run -i --rm quay.io/coreos/butane:release --strict < filename.bu > filename.ign
