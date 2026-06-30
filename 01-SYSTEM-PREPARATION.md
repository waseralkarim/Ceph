# Phase 1: System Preparation

Before we can run Ceph, we need to prepare the system with required dependencies and proper configuration.

## Prerequisites

- Ubuntu 22.04 LTS (or similar Debian-based Linux)
- Root or sudo access
- Stable network connection
- IP address: 192.168.16.51 (adjust as needed for your network)

## Step 1.1: Update System

```bash
sudo apt update && sudo apt upgrade -y
```

This ensures:
- All existing packages are up to date
- Security patches are applied
- No dependency conflicts with new packages

## Step 1.2: Install Required Dependencies

```bash
sudo apt install -y docker.io chrony lvm2
```

### What each package does:

- **docker.io** — Container runtime. Ceph runs in containers via `cephadm`
- **chrony** — NTP client for time synchronization. Critical for cluster coordination (daemons must have synchronized clocks)
- **lvm2** — Logical Volume Manager. Optional but useful for managing disks

## Step 1.3: Enable Services

```bash
sudo systemctl enable --now docker
sudo systemctl enable --now chrony
```

- `enable` — Start the service on boot
- `--now` — Start the service immediately

## Step 1.4: Verify Time Synchronization

```bash
chronyc tracking
```

**Expected output:** Should show status `SYNC` or similar. Example:

```
Reference ID    : 216.239.35.12 (time.google.com)
Stratum         : 2
Ref time (UTC)  : Mon Jan 01 12:34:56 2024
System time     : 0.000000000 seconds slow of NTP time
```

**Why this matters:** Ceph monitors rely on synchronized time to order cluster state changes. If time drifts by more than ~5 seconds, monitors may reject updates.

## Step 1.5: Verify Docker

```bash
docker --version
```

**Expected output:**
```
Docker version 20.10.x, build xxxxxxxx
```

## Step 1.6: Verify Python 3

```bash
python3 --version
```

**Expected output:**
```
Python 3.10.x
```

Ceph tools require Python 3 (typically 3.8+).

## Step 1.7: Add Hostname to /etc/hosts

```bash
echo "192.168.16.51 ceph-ssd" | sudo tee -a /etc/hosts
```

This creates a DNS entry so you can reference the host as `ceph-ssd` instead of `192.168.16.51`.

**Note:** Replace `192.168.16.51` with your actual IP address if different. Replace `ceph-ssd` with your hostname.

Verify it worked:

```bash
ping ceph-ssd
```

Should resolve to your IP address.

---

## Summary

After this phase, you should have:

✅ System fully updated
✅ Docker installed and running
✅ Time synchronized via NTP
✅ Hostname resolvable

**Next:** [Install cephadm](./02-INSTALL-CEPHADM.md)
