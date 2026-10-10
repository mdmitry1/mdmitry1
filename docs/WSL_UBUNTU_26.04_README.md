# WSL2 Configuration & Networking Setup Guide

This guide details how to configure WSL2 GUI display forwarding, enable mirrored networking mode, clean up obsolete port proxy rules, prevent SSH connection drops, and ensure seamless local loopback resolution using Windows hostnames.

---

## 1. Configure GUI Display Environment (Inside WSL)

Create the user environment configuration file so that systemd user services recognize your display server:

Create or edit `~/.config/environment.d/wsl.conf`:

```ini
DISPLAY=:0
WAYLAND_DISPLAY=wayland-0
XDG_CURRENT_DESKTOP=GNOME
```

---

## 2. Export Display Variables in Shell Profile (Inside WSL)

Ensure that display environment variables are exported and updated in the systemd/D-Bus user session (e.g. so GNOME Terminal hyperlinks and GUI apps always open properly).

Append the following block to your `~/.bashrc`:

```bash
export XDG_CURRENT_DESKTOP=GNOME

# Forward display to systemd user services so GNOME Terminal links always work
if [ -n "$DISPLAY" ]; then
    systemctl --user set-environment DISPLAY="$DISPLAY" 2>/dev/null
    systemctl --user set-environment WAYLAND_DISPLAY="$WAYLAND_DISPLAY" 2>/dev/null
    dbus-update-activation-environment --systemd DISPLAY WAYLAND_DISPLAY 2>/dev/null
fi
```

Apply the changes to your current shell:

```bash
source ~/.bashrc
```

---

## 3. Configure SSH Quality of Service (Inside WSL)

On some routers, VPNs, and virtual network adapters, the default Quality of Service (QoS) bits in SSH packets can cause hangs, timeouts, or dropped connections. Setting `IPQoS none` resolves these issues.

Create or edit `~/.ssh/config`:

```sshconfig
Host *
    IPQoS none
```

Ensure correct file permissions:

```bash
chmod 600 ~/.ssh/config
```

---

## 4. Enable Mirrored Networking in Windows

WSL2 mirrored networking mode shares the host's network interfaces directly with Linux, eliminating the need for manual port forwarding.

Create or edit `.wslconfig` in your Windows User profile directory (e.g., `C:\Users\<windows_username>\.wslconfig` or `/mnt/c/Users/<windows_username>/.wslconfig`):

> **Note:** Ensure the file uses DOS/Windows line endings (`CRLF`).

```ini
[wsl2]
networkingMode=mirrored
```

To apply changes, restart WSL from PowerShell:

```powershell
wsl --shutdown
```

---

## 5. Remove Obsolete Windows PortProxy Rules

With mirrored networking enabled, `netsh interface portproxy` rules are no longer needed and can block ports (e.g., port 8888 for Jupyter) on the Windows host.

Run the following commands in **PowerShell as Administrator**:

```powershell
# Remove portproxy rule on port 8888
netsh interface portproxy delete v4tov4 listenaddress=0.0.0.0 listenport=8888

# Restart IP Helper service to immediately release the port
Restart-Service iphlpsvc
```

---

## 6. Add IPv6 Loopback to Windows Hosts File

When connecting to services (like Jupyter) via the Windows machine hostname (`http://<hostname>:8888/`), Windows often prioritizes IPv6 queries. Add an IPv6 loopback entry to prevent connection timeouts.

Open Notepad as Administrator and edit:

```text
C:\Windows\System32\drivers\etc\hosts
```

Add the following entry (replacing `<hostname>` with your machine's computer name):

```text
::1       <hostname>
```

> **Tip:** If the change does not take effect immediately, flush the Windows DNS cache:
> ```cmd
> ipconfig /flushdns
> ```
> or clear your browser's internal DNS cache (`chrome://net-internals/#dns` or `edge://net-internals/#dns`).
