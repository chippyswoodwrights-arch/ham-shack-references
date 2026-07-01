# OpenWebRX Waterfall Display — Build Notes

## Hardware

| Part | Notes |
|---|---|
| Dell Wyse 3040 | Fanless x86_64 thin client, ~$40 refurbished (Tech For Less part 3YX35) |
| RTL-SDR Blog V4 | ~$35. Requires V4-specific drivers — do NOT use standard osmocom package |
| Nooelec Lana LNA (optional) | Use on VHF/UHF only (ADS-B 1090 MHz, NOAA 137 MHz). Skip for HF — V4 is sensitive enough |
| Antenna | Included dipole for initial test. DBJ-1 or discone for permanent install |

**One Wyse, one job.** OpenWebRX gets its own Wyse. Do not share with AllStar node.

---

## OS

**Debian 13 Trixie (amd64)**
- Download ISO: https://www.debian.org/releases/trixie/
- Flash with Rufus (rufus.ie) — same process as AllStar Wyse
- Wyse 3040 boots from USB. DisplayPort adapter required if no HDMI monitor available.

---

## Step 1 — RTL-SDR Blog V4 Drivers

The V4 requires drivers from the RTL-SDR Blog GitHub repo — NOT the standard Debian `rtl-sdr` package.
Using the wrong drivers causes no signals, wrong frequencies, or corrupted output.

**Primary source:** https://www.rtl-sdr.com/v4/ (verified 2026-06-23)

```bash
# Remove any existing RTL-SDR drivers
sudo apt purge ^librtlsdr
sudo rm -rvf /usr/lib/librtlsdr* /usr/include/rtl-sdr* \
  /usr/local/lib/librtlsdr* /usr/local/include/rtl-sdr* \
  /usr/local/include/rtl_* /usr/local/bin/rtl_*

# Install build dependencies
sudo apt-get install libusb-1.0-0-dev git cmake pkg-config build-essential

# Clone and build V4 drivers
git clone https://github.com/rtlsdrblog/rtl-sdr-blog
cd rtl-sdr-blog
mkdir build
cd build
cmake ../ -DINSTALL_UDEV_RULES=ON
make
sudo make install
sudo cp ../rtl-sdr.rules /etc/udev/rules.d/
sudo ldconfig

# Blacklist the DVB-T TV driver — critical, without this V4 won't function
echo 'blacklist dvb_usb_rtl28xxu' | sudo tee --append /etc/modprobe.d/blacklist-dvb_usb_rtl28xxu.conf

sudo reboot
```

**Verify after reboot:**
```bash
rtl_test -t
# Should show: "Found 1 device(s)" and tuner type E4000 or R820T2
```

---

## Step 2 — OpenWebRX+

Use the **OpenWebRX+** fork (luarvique), not the original openwebrx.de package.
OpenWebRX+ has active maintenance, Trixie support, and better decoder coverage.

**Primary source:** https://luarvique.github.io/ppa/ (verified 2026-06-23)

```bash
# Add OpenWebRX+ GPG key
curl -s https://luarvique.github.io/ppa/openwebrx-plus.gpg | \
  sudo gpg --yes --dearmor -o /etc/apt/trusted.gpg.d/openwebrx-plus.gpg

# Add Trixie repository
sudo tee /etc/apt/sources.list.d/openwebrx-plus.list \
  <<<"deb [signed-by=/etc/apt/trusted.gpg.d/openwebrx-plus.gpg] https://luarvique.github.io/ppa/trixie ./"

# Install
sudo apt update
sudo apt install openwebrx

# Enable and start
sudo systemctl enable --now openwebrx
```

**Access the interface:**
```
http://<wyse-ip>:8073
```

Default admin setup: http://127.0.0.1:8073/settings (first run wizard)

---

## Step 3 — Configure OpenWebRX+

### Add RTL-SDR V4 as SDR Device

In the web UI: Settings → SDR Devices → Add device

| Setting | Value |
|---|---|
| Type | RTL-SDR |
| Device string | rtl=0 |
| Sample rate | 2400000 (2.4 Msps) — good balance for Wyse 3040 CPU |
| PPM correction | 0 (calibrate later if needed) |

### Bias Tee (for Lana LNA)

If using the Nooelec Lana powered via bias tee:
- Device string: `rtl=0,bias_t=1`
- The V4 has a built-in bias tee — no separate USB cable needed for the Lana

### Profiles — Suggested Starting Points

| Profile | Frequency | Bandwidth | Use |
|---|---|---|---|
| 20m FT8 | 14.074 MHz | 96 kHz | HF monitoring (direct sampling) |
| 15m FT8 | 21.074 MHz | 96 kHz | HF monitoring (direct sampling) |
| 2m calling | 146.520 MHz | 1 MHz | Local FM/simplex |
| ADS-B | 1090 MHz | 2 MHz | Aircraft tracking |
| NOAA WX sat | 137.500 MHz | 1 MHz | Weather satellite |

**HF note:** V4 has direct HF sampling via Q-branch input. In OpenWebRX+, set device to
`rtl=0,direct_samp=3` for HF coverage without upconverter. Lana should NOT be in line for HF.

---

## Step 4 — Network Access

The waterfall is accessible from any browser on the local network at `http://10.0.0.xxx:8073`.

To find the Wyse IP after install:
```bash
ip addr show
```

Lock it to a static IP via router DHCP reservation (use MAC address shown in `ip addr`).

**Suggested IP assignment:** Reserve a static address for the OpenWebRX Wyse in router settings.
Document it in `Ham Shack\frequencies.md` under the station equipment table.

---

## Step 5 — Optional: SoapySDR (for additional decoders)

OpenWebRX+ can use SoapySDR for additional hardware support and some decoder integrations.
Not required for basic RTL-SDR V4 operation — add only if a specific decoder needs it.

```bash
sudo apt install soapysdr-tools libsoapysdr-dev
sudo apt install soapysdr-module-rtlsdr
```

**After installing SoapySDR RTL module, rebuild RTL-SDR Blog V4 drivers:**
```bash
cd ~/rtl-sdr-blog/build
make
sudo make install
sudo ldconfig
```
(The V4 guide notes SoapySDR users may need to reinstall drivers after adding SoapySDR.)

---

## Known Gotchas

| Issue | Fix |
|---|---|
| No signals / wrong frequency | Wrong driver. Must use rtlsdrblog/rtl-sdr-blog, not osmocom |
| Device not found | DVB-T module loaded. Check blacklist file, reboot |
| HF shows nothing | Add `direct_samp=3` to device string |
| Lana causing overload on HF | Remove Lana from HF chain — only use on VHF/UHF |
| CPU overload on Wyse 3040 | Lower sample rate to 1.024 Msps, or reduce active profiles |
| Can't reach UI from other devices | Check `systemctl status openwebrx`, check firewall (`ufw status`) |

---

## Sources (verified 2026-06-23)

- RTL-SDR Blog V4 Users Guide: https://www.rtl-sdr.com/v4/
- OpenWebRX+ PPA (luarvique): https://luarvique.github.io/ppa/
- OpenWebRX+ Home Page: https://fms.komkon.org/OWRX/
- fluxcoil Debian + RTL-SDR + OpenWebRX+ walkthrough: https://blog.fluxcoil.net/posts/2025/09/debian-with-rtl-sdr-and-openwebrx-setup/
- OpenWebRX+ GitHub: https://github.com/luarvique/ppa

---

## Status

- [ ] Wyse 3040 — Debian 13 Trixie installed
- [ ] RTL-SDR Blog V4 drivers installed and verified (`rtl_test -t`)
- [ ] OpenWebRX+ installed, service running
- [ ] RTL-SDR V4 device added in UI
- [ ] At least one profile configured and waterfall visible
- [ ] Static IP reserved in router, documented in frequencies.md
