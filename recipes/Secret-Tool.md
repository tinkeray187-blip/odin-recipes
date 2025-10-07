# Wi-Fi Secret Extraction Toolkit

> WARNING: These tools can reveal stored Wi-Fi passwords and other secrets. Use **only** on machines you own or administer. Do **not** use them to access networks or systems you are not authorized to access.

This repository includes multiple tools and approaches to enumerate Wi-Fi connections and attempt to retrieve stored PSKs/passwords on Windows and Linux (NetworkManager-based distributions). Each tool targets a different storage location or method:

- `windows_wifi_extract.py` — Windows `netsh` based extractor (works on Windows only).
- `get_wifi_passwords.py` — Python parser for `/etc/NetworkManager/system-connections/` files and `/etc/wpa_supplicant/wpa_supplicant.conf`.
- `nm_keyring_super.py` — Full-featured "steroids" tool: tries Secret Service (GNOME Keyring / libsecret) via `secretstorage`, then `nmcli`, then system files.
- `nm_shell_keyring.sh` — Shell-tool alternative using `nmcli` and `secret-tool` (no Python venv required).

---

## Quick summary of what each tool does



### 1) `nm_keyring_super.py` — The most powerful (recommended starting point on Linux)
- **Purpose:** Try every reasonable way to retrieve Wi-Fi PSKs on a Linux desktop:
  1. Query Secret Service (GNOME Keyring / libsecret) using `secretstorage` (preferred).
  2. Query `nmcli` for stored PSKs (if NetworkManager exposes them).
  3. Parse `/etc/NetworkManager/system-connections/*` files (requires root to read many of these).
  4. Parse `/etc/wpa_supplicant/wpa_supplicant.conf` as an extra fallback.
- **How it matches secrets:** Heuristic matching by connection name, UUID, SSID and common attribute keys (e.g., `connection.id`, `connection.uuid`, `ssid`).
- **Run as:** the logged-in user (do **not** run with `sudo` — Secret Service is per-user via DBUS).
- **Best for:** Desktop systems using NetworkManager with secrets stored in the user keyring.



### 2) `get_wifi_passwords.py` — Direct file parser (simple, robust)
- **Purpose:** Parse NetworkManager connection files and wpa_supplicant file for SSID → PSK pairs.
- **Run as:** `sudo` if you want to read `/etc/NetworkManager/system-connections/*` (these files are often root-owned).
- **Best for:** Servers or systems where secrets are stored in plaintext in system-connections files or wpa_supplicant.



### 3) `nm_shell_keyring.sh` — Shell-tool lightweight approach
- **Purpose:** Use `nmcli` and `secret-tool` (from `libsecret-tools`) in a compact bash script to try:
  1. `nmcli` PSK
  2. `secret-tool lookup` with common attribute names
  3. Read system-connections files when readable
- **Run as:** logged-in user (no venv, no Python). Do **not** `sudo` this script (you’ll lose DBUS session).
- **Best for:** Quick runs on a workstation where you prefer shell commands.



### 4) `windows_wifi_extract.py` — Windows `netsh` extractor
- **Purpose:** Run `netsh wlan show profiles` and then `netsh wlan show profile "<name>" key=clear` to get saved SSIDs and passwords.
- **Run as:** On Windows (cmd/powershell) with Python installed.
- **Best for:** Windows administrators.

---

## Dependencies & install

A convenience script `install.sh` is included for Debian/Parrot-style systems. It installs system packages and Python dependencies described below.

**Linux (Debian/Parrot)**
- System packages: `network-manager`, `libsecret-tools`, `python3-pip`, `python3-venv`, `build-essential`, `python3-dev`, `libdbus-1-dev`, `libdbus-glib-1-dev`
- Python packages: `secretstorage`, `dbus-python`

**Windows**
- Python 3.x (no extra packages required for the simple `netsh` extractor).

See `install.sh` for one-line install commands.

---

## How to run (examples)

### Run the super tool (recommended on Linux desktop)
1. Ensure Python deps installed (see `install.sh`).
2. From your logged-in desktop terminal (do **not** `sudo`):
   ```bash
   python3 nm_keyring_super.py
