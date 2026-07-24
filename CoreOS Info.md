- To keep up with the latest and greatest Butane version: https://coreos.github.io/butane/specs/

- This is what got the flatpak version working:

chris@chris-desktop:~/Downloads/coreos$ ssh-add -l
The agent has no identities.
chris@chris-desktop:~/Downloads/coreos$ mkdir -p ~/.bitwarden
chris@chris-desktop:~/Downloads/coreos$ ln -sf /home/chris/.var/app/com.bitwarden.desktop/data/.bitwarden-ssh-agent.sock /home/chris/.bitwarden/ssh-agent.sock
chris@chris-desktop:~/Downloads/coreos$ export SSH_AUTH_SOCK="/home/chris/.bitwarden/ssh-agent.sock"
chris@chris-desktop:~/Downloads/coreos$ ssh-add -l
Error connecting to agent: Connection refused
chris@chris-desktop:~/Downloads/coreos$ ssh-add -l
256 SHA256:aQZ5+UyIQku7OPvN9jbal4f6Cu8YD1zNgD7DORTf/H0 test-ssh-key (ED25519)
256 SHA256:QQbielzzHgA5iEnGtDc/HM34NTH74XT/E3cK4ycXx9I coreos-vm (ED25519)
chris@chris-desktop:~/Downloads/coreos$ ssh core@192.168.122.232
Fedora CoreOS 44.20260707.3.1
Tracker: https://github.com/coreos/fedora-coreos-tracker
Discuss: https://discussion.fedoraproject.org/tag/coreos




# Comprehensive Guide: Using Flatpak Bitwarden SSH Agent with Fedora CoreOS

This guide explains how to use the Flatpak version of Bitwarden as an SSH Agent to securely provision and authenticate with Fedora CoreOS (FCOS).

---

## 1. The CoreOS Boot Logic

1. **Butane File (`.bu`)**: You embed your **Public Key** text string into the Butane configuration.
2. **Ignition File (`.ign`)**: You transpile the Butane file into an Ignition JSON file.
3. **First Boot**: The Fedora CoreOS installer reads the Ignition file **exactly once** on the first boot and injects the public key into the default `core` user's `authorized_keys` file.
4. **Authentication**: When you SSH into the machine, your local SSH client handles the private key challenges via the Bitwarden SSH Agent socket. **The private key never leaves your vault.**

---

## 2. Configuring the Flatpak Bitwarden Socket Fix

Because Flatpak runs Bitwarden inside a secure sandbox, it cannot write to the standard `~/.bitwarden/` directory. It writes deep inside its isolated app-data runtime path instead. 

Follow these steps to safely link the sandboxed socket to your host system:

### Step 1: Create a Standard Directory Structure
Create the native directory path that system tools expect to find:
```bash
mkdir -p ~/.bitwarden
```

### Step 2: Create a Symlink from the Sandbox to the Host
Link the hidden Flatpak socket file directly to your newly created folder:
```bash
ln -sf /home/$USER/.var/app/com.bitwarden.desktop/data/.bitwarden-ssh-agent.sock ~/.bitwarden/ssh-agent.sock
```
*(Note: If the file is missing, open Bitwarden -> Go to Settings -> Developer -> Toggle "Enable SSH Agent" off and back on to force-generate the socket).*

### Step 3: Make the Environment Variable Permanent
The `ssh-add` utility relies on the `SSH_AUTH_SOCK` environment variable to locate active keys. Append this variable permanently to your shell profile:

```bash
# For Bash users:
echo 'export SSH_AUTH_SOCK="$HOME/.bitwarden/ssh-agent.sock"' >> ~/.bashrc
source ~/.bashrc

# For Zsh users:
echo 'export SSH_AUTH_SOCK="$HOME/.bitwarden/ssh-agent.sock"' >> ~/.zshrc
source ~/.zshrc
```

### Step 4: Configure the SSH Client
Update your SSH client configuration file to point to the clean symlinked socket path.

Open or create `~/.ssh/config`:
```bash
nano ~/.ssh/config
```

Add the following block (**Do not put quotes around the path**, as OpenSSH treats quoted tildes as literal text strings):
```text
Host *
  IdentityAgent ~/.bitwarden/ssh-agent.sock
```

---

## 3. Verifying the Connection Pipeline

Before deploying CoreOS, always verify your local environment can see the keys inside your vault.

1. Open your **Bitwarden Desktop App** and completely **Unlock your Vault** (keys are strictly hidden when locked).
2. Ensure your key is saved explicitly as an **"SSH Key" item type** inside your **Personal Vault** (the agent cannot read items saved in Organization Vaults or standard Login text fields).
3. Open a fresh terminal window and check available identities:
   ```bash
   ssh-add -l
   ```

**Expected Successful Output:**
```text
256 SHA256:abc123xyz... coreos-key (ED25519)
```
*(If it says "The agent has no identities", double-check that your vault is fully unlocked and that the item type in Bitwarden is explicitly set to "SSH Key").*

---

## 4. Writing the Butane File & Deployment

### Step 1: Add the Key to Butane (`coreos.bu`)
Paste your copied Bitwarden public key into your configuration schema:

```yaml
variant: fcos
version: 1.7.0  # Check the link for the latest and greatest version
passwd:
  users:
    - name: core
      ssh_authorized_keys:
        - "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI...your_copied_bitwarden_public_key_string..."
```

### Step 2: Transpile to Ignition
Convert your layout into JSON format using the local binary or a mounted container directory:

```bash
podman run -i --rm quay.io/coreos/butane:release --strict < coreos.bu > coreos.ign
```

### Step 3: Accessing the Provisioned Machine
Boot your target infrastructure using the generated `coreos.ign` file. Once the OS finishes initialization, connect securely as the default `core` user:

```bash
ssh core@<your-coreos-ip>
```
A Bitwarden confirmation notification will appear natively on your host desktop. Click **Approve** to securely sign the transaction and gain access!
