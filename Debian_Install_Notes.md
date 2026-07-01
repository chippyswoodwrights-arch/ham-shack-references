# Debian 13 Trixie Install Notes — Wyse 3040 and Similar Hardware

*Shared reference for all projects. Applies to: AllStar node, OctoPrint, Pi-hole, Frigate, any Wyse/thin-client Debian build.*
*Last updated: 2026-06-30*

---

## Boot USB — Create Once, Reuse Forever

- ISO: Debian 13 Trixie amd64 netinst — https://www.debian.org/releases/trixie/
- Tool: Rufus, ISO Image mode (recommended when prompted)
- When Rufus asks about Syslinux download: click **No**
- When asked ISOHybrid mode: **ISO Image mode**
- After Rufus finishes: copy updated preseed.cfg to root of USB before ejecting

**The preseed.cfg on the USB automatically handles:**
- SSH password auth enabled (sed replace, not append)
- PermitRootLogin yes
- sudo installed
- Passwordless sudo for matt
- Passwords set: root and matt both `octoprint123`
- Default SSH key installed (`wyse_default`) — Claude can SSH in immediately after boot
- Hostname defaults to `wyse` — change per device after install (see Post-Install)

---

## Default SSH Key

The preseed installs a default keypair so Claude can take over immediately after first boot.

- Private key: `~/.ssh/wyse_default` (on Windows PC)
- Public key: `~/.ssh/wyse_default.pub`
- Comment: `wyse-default-key`

**This is the "factory default" key — same concept as a router's default password.**
Replace with a device-specific key after setup if sharing the repo publicly.

To SSH into any fresh Wyse build:
```bash
ssh -i ~/.ssh/wyse_default matt@<ip>
```

---

## Pre-Install Checklist

Before starting any Wyse install, have these on hand:
- USB keyboard (can't borrow the laptop keyboard mid-install)
- Monitor with HDMI input + HDMI cable
- DisplayPort to HDMI adapter (Plugable DP-HDMI) — Wyse has DisplayPort out only
- Ethernet cable confirmed to be a **data** cable (not charge-only)
- Boot USB with preseed.cfg (see above)

Confirm ethernet is a real data cable first — a dud cable means no network mirror, no SSH server, stuck typing everything on the TV.

---

## Install Procedure

1. Plug in keyboard, monitor, ethernet, boot USB
2. Power on — tap F12 for boot menu, select USB
3. Select **Graphical install**
4. Follow prompts — blow past hostname prompt (preseed sets it to `wyse`), leave domain blank
5. Partition: **entire disk**, all files in one partition
6. Mirror: accept it, use `deb.debian.org`, leave proxy blank
7. Software: SSH server + standard system utilities only — no desktop
8. Pull USB when reboot starts
9. Machine boots — Claude can SSH in immediately with `wyse_default` key

---

## Post-Install — Per Device Customization

**Do these after every build before handing to Claude:**

Find the IP from your router's DHCP table, then:

```bash
# Set hostname
sudo hostnamectl set-hostname <device-name>

# Change passwords
passwd
sudo passwd root
```

Claude handles everything else from here via SSH with the default key.

---

## Known Gotchas

### SSH password auth disabled by default
Debian 13 ships with `PasswordAuthentication no` explicitly set. SSH uses the **first matching value** — appending `yes` at the end does NOT override it.

**Wrong:**
```bash
echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config
```

**Correct:**
```bash
sed -i 's/^#\?PasswordAuthentication.*/PasswordAuthentication yes/' /etc/ssh/sshd_config
systemctl restart ssh
```

The preseed late_command now does this automatically. If it fails for any reason, use the sed command above manually as root.

### PAM faillock — account lockout after repeated SSH failures
After many failed SSH attempts, PAM locks the account. Symptoms: correct password rejected via SSH but works at console.

Fix (run as root on the Wyse):
```bash
faillock --user matt --reset
```

### apt sources format changed in Debian 13
Old guides show `/etc/apt/sources.list`. Debian 13 uses `/etc/apt/sources.list.d/debian.sources`.
If apt is broken after install, check that file exists and has correct content.

### Ethernet cable must be a data cable
A charge-only ethernet cable shows both interfaces DOWN in `ip addr`. Swap with a known-good cable. Learned this the hard way on the AllStar node build.

### Wyse 3040 BIOS
- Boot menu: **F12** at power on
- BIOS setup: **F2** at power on
- May have admin password set by previous owner — try `Fireport` as default
- USB must be in boot sequence to appear as F12 option
- UEFI boot only — ensure USB was written in ISO Image mode by Rufus

### USB autosuspend kills keyboard at login
Debian's USB autosuspend can kill the keyboard at the login prompt. If keyboard dies mid-login, boot GRUB recovery mode (hold Shift at power on → Advanced → recovery → root shell) to access root without keyboard.

### preseed.cfg is read-only on the USB after Rufus writes
The ISO filesystem is read-only — cannot edit preseed.cfg in place. Workflow:
1. Edit preseed.cfg on desktop
2. Run Rufus to rewrite the USB with the Debian ISO
3. After Rufus finishes, copy updated preseed.cfg to root of USB before ejecting

---

## Deployed Instances

| Hostname | IP | Project | SSH Key | Notes |
|---|---|---|---|---|
| allstarnode | 10.0.0.196 | AllStar node | ~/.ssh/wyse_key | user=matt |
| octoprint | 10.0.0.170 | OctoPrint | ~/.ssh/wyse_octoprint | user=matt, OctoPrint at :5000 |
| pihole | 10.0.0.191 | Pi-hole | ~/.ssh/wyse_octoprint | user=matt, Pi-hole admin at :80/admin |

---

## Preseed.cfg — Current Version on USB

Located at root of boot USB. Master copy: `C:\Users\mattc\OneDrive\Desktop\Ham Shack\preseed.cfg` (also in git when Ham Shack repo is set up).

Key late_command additions (2026-06-30):
- Default SSH key (`wyse_default`) installed to `/home/matt/.ssh/authorized_keys`
- Hostname set to `wyse` by default
- All prior SSH fixes retained (sed replace, PermitRootLogin, sudoers)
