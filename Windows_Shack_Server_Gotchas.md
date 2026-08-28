# Windows Shack Server Build — Known Gotchas

Compiled from KO6NOI's OptiPlex 3070 Micro build (Windows 11 Pro shack server: Ham AI
assistant, OpenHamClock, Pi-hole, remote access). Applies to any similar build — a small
Windows PC set up as an always-on ham shack server with remote access.

**Purpose:** these are real problems hit and solved, not theoretical advice. If your Claude
session hits one of these, point it here first before troubleshooting from scratch.

---

## Remote Access

### RDP disconnects the local console session (TV goes blank)
**Symptom:** connecting via Remote Desktop from another machine blanks whatever's on the
physical monitor/TV plugged into the server.

**Cause:** confirmed against Microsoft Learn docs — this is standard Windows client-edition
behavior, not a bug. Windows Home/Pro (non-Server) editions support only one interactive
session at a time. RDP takes over the console session rather than adding a second one.

**Fix:** install a VNC server instead (TightVNC is free and simple) for any use case where
the physical display needs to stay live while you also connect remotely. VNC shares the
existing session instead of taking it over.

### Microsoft-account RDP login keeps failing ("logon attempt failed")
**Symptom:** RDP rejects credentials that work fine for signing into the machine locally.

**Cause:** never fully pinned down — likely a PIN vs. full-password mismatch, since Windows
Hello lets you sign in locally with a PIN that isn't the same as the full account password
RDP requires.

**Fix:** create a local (non-Microsoft) admin account specifically for RDP. Sidesteps the
whole problem and is more reliable for a headless server anyway — no dependency on Microsoft
cloud auth for local network access.

**Gotcha within the fix:** a brand-new local account must sign in **locally at least once**
(interactively, or via VNC) before RDP will accept it — Windows hasn't finished building the
user profile until that first interactive logon happens. If RDP rejects a freshly created
account with "unknown username or bad password" even though the credentials are correct,
this is almost certainly why.

### Sleep breaks remote access entirely
Set Screen and Sleep to **Never** (when plugged in) before relying on RDP/VNC. A sleeping
Windows client-edition machine cannot be woken by an incoming RDP/VNC connection — you have
to physically walk over and wake it, which defeats the purpose of a remote server.

---

## OpenSSH Server Setup

### `Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0` hangs indefinitely
The PowerShell cmdlet wrapper can hang forever with no error and no progress.

**Fix:** run the equivalent DISM command directly instead — it succeeded both times the
PowerShell cmdlet hung:
```
DISM /Online /Add-Capability /CapabilityName:OpenSSH.Server~~~~0.0.1.0
```
DISM shows a real percentage progress bar, so you can actually tell if it's working versus
frozen. If DISM also hangs, check Windows Update service status
(`Get-Service wuauserv`) — capability installs route through it even for "online" DISM ops.

### Key-based auth for an admin account needs a specific file
Regular users' authorized_keys goes in `.ssh\authorized_keys` in their home folder — but
for accounts in the **Administrators** group, Windows OpenSSH uses a separate system-wide
file instead:
```
%ProgramData%\ssh\administrators_authorized_keys
```
Lock down its permissions after adding a key or SSH will refuse to use it:
```powershell
icacls "$env:ProgramData\ssh\administrators_authorized_keys" /inheritance:r
icacls "$env:ProgramData\ssh\administrators_authorized_keys" /grant "Administrators:F" "SYSTEM:F"
```
Then `Restart-Service sshd` to pick up the change.

---

## WSL2 + Docker Desktop

### `wsl --install` / `wsl --update` fail with a misleading "not installed" message
Even after confirming both required Windows features are enabled, `wsl.exe` keeps printing
the generic "WSL is not installed, run wsl.exe --install" message for every subcommand.

**Cause:** the two prerequisite Windows optional features need to be enabled *and the
machine rebooted* before `wsl.exe` itself will do anything useful:
```
dism /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
Reboot after both. If `wsl --update` still fails afterward (can happen if it can't reach the
Microsoft Store backend from a script/service context), download the WSL2 kernel package
directly from the project's GitHub releases and install the MSI manually:
```
https://github.com/microsoft/WSL/releases/latest
```
(Check the API for the current filename — the version number changes — rather than guessing
a specific version in the URL, which will 404.)

### `docker pull` fails over SSH: "specified logon session does not exist"
**Symptom:** `docker pull <image>` fails with a credential helper error when run over an
SSH session, even for public images that don't need authentication, even after removing
`credsStore` from `~/.docker/config.json`.

**Cause:** this is a known, documented limitation
([docker/cli#4353](https://github.com/docker/cli/issues/4353),
[docker/for-win#12888](https://github.com/docker/for-win/issues/12888)) — Docker Desktop's
credential helper on Windows needs an interactive desktop logon token to function, which an
SSH (service-context) session does not have. No config change fixes this; it's a Windows
session-type limitation, not a Docker bug you can configure around.

**Fix:** run `docker pull` (and `docker login`) from an actual interactive session — sit at
the machine, or drive it via VNC. Once the image is pulled locally, everything else
(`docker run`, `docker exec`, `docker cp`, container management) works completely fine over
a plain SSH session — only the initial registry pull/auth needs the interactive session.

### Windows Firewall silently blocks other devices from reaching a container
After starting a container that serves something to the LAN (Pi-hole's DNS port, a web UI,
etc.), Windows may prompt "Do you want to allow public and private networks to access this
app?" for **Docker Desktop Backend**. If you miss or cancel this prompt, the container works
fine from the host itself but nothing else on the network can reach it. Click **Allow**.

---

## DNS / Networking

### Manually setting IPv4 DNS doesn't work — Windows still uses the old resolver
**Symptom:** you set a manual IPv4 DNS server (e.g. pointing at a Pi-hole), verify it saved
correctly, but `nslookup` still shows the old DNS server responding.

**Cause:** Windows will prefer **IPv6** DNS over IPv4 when both are present and IPv6 is
auto-configured (which it usually is on a home network via router advertisements/DHCPv6).
Setting only the IPv4 DNS server leaves the IPv6 resolver untouched and still pointing at
whatever your ISP assigned. Confirm with:
```powershell
Get-DnsClientServerAddress | Format-Table InterfaceAlias, AddressFamily, ServerAddresses
```
AddressFamily `2` is IPv4, `23` is IPv6 — check both.

**Simplest fix:** disable IPv6 on the adapter entirely (uncheck "Internet Protocol Version 6
(TCP/IPv6)" in adapter properties). No real downside on a home network. Alternative if you
want to keep IPv6: set the IPv6 DNS server explicitly too, via elevated PowerShell —
`Set-DnsClientServerAddress` needs admin rights, it will fail silently otherwise
("Access to a CIM resource was not available to the client").

**Also disable DNS over HTTPS (DoH)** if you're pointing DNS at a local ad-blocker/filter —
DoH bypasses your manually configured DNS server entirely and routes to whatever provider
the browser/OS has hardcoded, defeating the whole point.

### Xfinity gateways may not expose a custom DNS override at all
Confirmed on an XB10: neither the xFi mobile app nor the local admin panel
(`10.0.0.1` → Gateway → Connection → Local IP Network) has an editable DNS field — only a
read-only display of Comcast's own DNS servers (75.75.75.75 / 75.75.76.76). This appears to
be true of most current-generation xFi gateways, not just this model.

**If you hit this:** there is no whole-house fix available through the stock Xfinity gateway.
Either configure DNS per-device manually (tedious but works, start with the devices you
actually use daily), or put the gateway in bridge mode and add your own router behind it
(bigger project, full control).

---

## Git

### A PDF (or other large binary) accidentally committed early in history blocks pushes
**Symptom:** `git push` fails with something like `unable to read <hash>` /
`remote end hung up unexpectedly`, and `git fsck --full` shows `missing blob` errors tied to
a tree in your commit history.

**Cause:** a large binary file (PDF manual, etc.) that was committed at some point and later
`.gitignore`'d has a corrupted or missing object in the local repo, even though the file
itself isn't tracked anymore.

**Fix:** strip the file from the entire history with `git filter-branch` (or `git-filter-repo`
if installed) rather than just deleting it from the current commit:
```bash
git filter-branch --force --index-filter \
  'git rm --cached --ignore-unmatch "path/to/file.pdf"' \
  --prune-empty -- --all
rm -rf .git/refs/original/
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git fsck --full   # should come back clean
```
This rewrites all commit hashes — fine for a solo local repo, but flag it before doing this
on anything already shared/cloned elsewhere.

---

## Misc

### A project's launch script/docs claim the wrong port
Don't trust a README or `start.bat`'s printed URL at face value if something "won't load" —
check the actual `.env` or config file for the real configured port before assuming the
service is broken. (OpenHamClock's bundled `start.bat` said `localhost:3000`; the real port
per its own `.env` was `3001`. The server was healthy the entire time — nothing was actually
hung, the browser was just pointed at the wrong port.)

Quick check from an SSH session, no GUI needed:
```
netstat -ano | findstr :<port>
```
and confirm something's actually `LISTENING` before troubleshooting further.

---

*Compiled 2026-08-28 from the KO6NOI Ham AI Assistant OptiPlex build session. See
`Ham Radio AI Assistant` repo (private) for the full build log if more context is needed.*
