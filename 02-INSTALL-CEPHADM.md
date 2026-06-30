# Phase 2: Install cephadm

`cephadm` is the Ceph cluster manager. It containerizes all Ceph daemons and manages their lifecycle on your cluster.

## What is cephadm?

- **Cluster bootstrap and deployment tool**
- **Container orchestrator** for Ceph (similar to what Docker Compose does)
- **Administrative interface** for managing Ceph services
- **Modern approach** recommended by Ceph (replaces older manual installation methods)

## Step 2.1: Add Ceph Reef Repository

### Why Reef?

Reef is the latest stable Ceph release (version 18.x). We're using Ubuntu 22.04 LTS (jammy), but we need to work around a bootstrap compatibility issue with Ubuntu 24.04 (noble).

### Add GPG Key

```bash
wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo tee /etc/apt/trusted.gpg.d/ceph.asc > /dev/null
```

This imports Ceph's GPG signing key so package signatures can be verified.

### Add Repository

```bash
echo "deb https://download.ceph.com/debian-reef/ jammy main" | sudo tee /etc/apt/sources.list.d/ceph.list
```

This adds the Ceph Reef repository for Ubuntu 22.04 LTS (jammy).

### Update Package List

```bash
sudo apt update
```

Fetch the latest package list from the new repository.

---

## Step 2.2: Download cephadm Binary via curl

### Why curl instead of apt?

The apt-installed `cephadm=18.2.7-1jammy` has a broken Python interpreter issue. The binary downloaded via curl works correctly.

### Download and Set Executable

```bash
CEPH_RELEASE=18.2.4
curl --silent --remote-name --location https://download.ceph.com/rpm-${CEPH_RELEASE}/el9/noarch/cephadm
chmod +x cephadm
```

This:
1. Sets `CEPH_RELEASE=18.2.4`
2. Downloads the `cephadm` binary (from Enterprise Linux 9 repo, but it's distribution-agnostic)
3. Makes it executable (`chmod +x`)

**Why EL9 binary on Ubuntu?** The binary itself doesn't depend on the OS distribution — it's a Python script that works on any Linux system with Python 3.

---

## Step 2.3: Install cephadm to System Path

```bash
sudo cp ./cephadm /usr/sbin/cephadm
```

Copies the binary to system path so it's available globally.

### Verify Installation

```bash
cephadm version
```

**Expected output:**
```
cephadm version 18.2.4 (...) reef (stable)
```

If you see something like:
```
cephadm version 18.2.7 ...
```

This means the apt-installed version is still being used. Run:

```bash
which cephadm
# Should show: /usr/sbin/cephadm
```

If it shows `/usr/bin/cephadm` or another location, remove the broken version:

```bash
sudo rm /usr/bin/cephadm
sudo cp ./cephadm /usr/sbin/cephadm
sudo update-alternatives --install /usr/bin/cephadm cephadm /usr/sbin/cephadm 100
```

---

## Summary

After this phase, you should have:

✅ Ceph Reef repository added
✅ cephadm 18.2.4 binary installed and accessible
✅ Verified with `cephadm version`

**Next:** [Bootstrap the Cluster](./03-BOOTSTRAP-CLUSTER.md)
