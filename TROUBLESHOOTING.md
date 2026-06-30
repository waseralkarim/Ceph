# Troubleshooting Guide

Common issues and solutions when setting up Ceph.

## General Health Checks

### Cluster Health is HEALTH_WARN

```bash
sudo ceph health detail
```

**Common causes:**

#### No OSDs Available
- **Issue:** `OSD count 0, should be >= 1`
- **Solution:** Follow [Phase 5: Add OSD](./05-ADD-OSD.md)

#### Low PG Count
- **Issue:** `Low PG in some pool`
- **Solution:** Increase PGs:
  ```bash
  sudo ceph osd pool set pool-name pg_num 64
  sudo ceph osd pool set pool-name pgp_num 64
  ```

#### Slow Operations
- **Issue:** `requests are blocked > X seconds`
- **Solution:**
  - Check OSD load: `sudo ceph osd status`
  - Check disk I/O: `iostat -x 1`
  - Check network: `iftop` or `nethogs`

#### PG is Down
- **Issue:** `Degraded object`
- **Solution:**
  ```bash
  sudo ceph pg stat  # See which PGs are down
  sudo ceph pg recover 1.2c  # Recover specific PG
  ```

---

## OSD Issues

### OSD is Down

```bash
sudo ceph osd tree
# Look for status != "up"
```

**Troubleshooting:**

```bash
# Check if OSD container is running
sudo ceph orch ps | grep osd

# View OSD logs
sudo ceph orch logs osd.0

# Restart OSD
sudo ceph orch daemon restart osd.0

# Check disk status
lsblk
sudo smartctl -a /dev/vdb  # If smartctl available
```

### OSD is In but Down

```bash
# Mark OSD back in (if it was marked out)
sudo ceph osd in 0

# Or manually recover
sudo ceph osd up 0
```

### OSD Memory Usage Too High

```bash
# Check current usage
sudo ceph osd status

# Set memory target
sudo ceph config set osd.0 osd_memory_target 4294967296  # 4GB
```

### Disk Capacity Full

```bash
# Check usage
sudo ceph df

# If OSD disk is full:
# 1. Add more OSDs
# 2. Delete unnecessary data
# 3. Reweight OSDs
sudo ceph osd reweight 0 0.5  # Reduce to 50%
```

---

## Pool Issues

### Pool Creation Failed

```bash
# Check error
sudo ceph osd pool ls

# Try again with more detail
sudo ceph osd pool create my-pool 32 --debug
```

**Common issue:** Not enough OSDs
- **Solution:** Must have >= number of replicas in OSD count

### Object Count Increasing But No Data Added

```bash
# Check PG status
sudo ceph pg stat

# Check for stale data
sudo ceph pg dump | grep stale
```

### Pool Replica Mismatch

```bash
# Check current settings
sudo ceph osd pool get pool-name size
sudo ceph osd pool get pool-name min_size

# For single OSD:
sudo ceph osd pool set pool-name size 1 --yes-i-really-mean-it
sudo ceph osd pool set pool-name min_size 1
```

---

## RGW Issues

### RGW Daemon Won't Start

```bash
# Check logs
sudo ceph orch logs rgw.default.ceph-ssd.xxx

# Check if realm exists
sudo radosgw-admin realm list

# Recreate realm if needed
sudo radosgw-admin realm create --rgw-realm=default --default
sudo radosgw-admin zonegroup create --rgw-zonegroup=default --master --default
sudo radosgw-admin zone create --rgw-zonegroup=default --rgw-zone=default --master --default
sudo radosgw-admin period update --commit
```

### S3 Endpoint Unreachable

```bash
# Test connectivity
curl http://localhost:8080

# Check firewall
sudo ufw status
# If needed: sudo ufw allow 8080

# Check if daemon is running
sudo ceph orch ps | grep rgw
```

### S3 Authentication Failed

```bash
# Verify user exists
sudo radosgw-admin user info --uid=myuser

# Recreate user
sudo radosgw-admin user create --uid=myuser --display-name="My User"

# Check buckets
sudo radosgw-admin bucket list
```

### RGW Pools Not Created

```bash
# Check what pools exist
sudo ceph osd pool ls

# Look for default.rgw.* pools

# If missing, try redeploying RGW:
sudo ceph orch rm rgw.default
sudo ceph orch apply rgw default default --placement="1 ceph-ssd" --port=8080
```

---

## CephFS Issues

### MDS Daemon Won't Start

```bash
# Check logs
sudo ceph orch logs mds.cephfs.ceph-ssd.xxx

# Check if filesystem exists
sudo ceph fs ls

# Check cluster status
sudo ceph -s | grep mds
```

**Solution:**

```bash
# If filesystem corrupt, remove and recreate
sudo ceph fs rm cephfs --yes-i-really-mean-it
sudo ceph fs volume create cephfs --placement="1 ceph-ssd"
```

### CephFS Mount Fails

```bash
# Verify filesystem exists
sudo ceph fs status

# Check if MDS is active
sudo ceph fs status | grep RANK

# Check mount error
sudo mount -t ceph ceph-ssd:/ /mnt/cephfs -o name=admin,secret=$(sudo ceph auth get-key client.admin)

# If permission denied, check auth
sudo ceph auth list | grep admin
```

### Cannot Write to CephFS

```bash
# Check filesystem status
sudo ceph fs status

# Check MDS rank
sudo ceph mds stat

# Try remounting with different options
sudo umount /mnt/cephfs
sudo mount -t ceph ceph-ssd:/ /mnt/cephfs -o name=admin,secret=$(sudo ceph auth get-key client.admin),relatime
```

### CephFS Mounting Hangs

```bash
# The issue is usually network or MDS down

# Check MDS status
sudo ceph mds stat

# Check network connectivity
ping -c 1 ceph-ssd

# Try with timeout
sudo timeout 10 mount -t ceph ceph-ssd:/ /mnt/cephfs -o name=admin,secret=$(sudo ceph auth get-key client.admin)
```

---

## RBD Issues

### Cannot Map RBD Image

```bash
# Check if image exists
sudo rbd ls -p rbd-pool

# Check image info
sudo rbd info rbd-pool/my-image

# Try mapping with details
sudo rbd map rbd-pool/my-image --debug --verbose
```

**Common issue:** RBD kernel module not loaded

```bash
# Load module
sudo modprobe rbd

# Verify
lsmod | grep rbd
```

### RBD Device Full

```bash
# Check usage
df -h /mnt/rbd-test

# Resize image (must be unmounted)
sudo umount /mnt/rbd-test
sudo rbd resize rbd-pool/my-image --size 10G
sudo mount /dev/rbd0 /mnt/rbd-test
```

### RBD Snapshot Issues

```bash
# List snapshots
sudo rbd snap ls rbd-pool/my-image

# Cannot delete due to dependents
# First, delete clones that depend on it
sudo rbd rm rbd-pool/my-image-clone
sudo rbd snap rm rbd-pool/my-image@snapshot1
```

### RBD Device Becomes Stale

```bash
# Force unmap
sudo rbd unmap /dev/rbd0 --force

# Check for stale mappings
sudo rbd showmapped

# Remap if needed
sudo rbd map rbd-pool/my-image
```

---

## Monitor Issues

### Monitor Down

```bash
# Check monitor status
sudo ceph mon stat

# Check monitor quorum
sudo ceph quorum_status

# View logs
sudo ceph orch logs mon.ceph-ssd

# Restart monitor
sudo ceph orch daemon restart mon.ceph-ssd
```

### No Quorum (Emergency!)

**This is serious.** Monitors are down and cluster cannot operate.

```bash
# Try to restart all monitors
sudo ceph orch restart mon

# If that doesn't work, force a new quorum:
# ⚠️  WARNING: This is destructive!
# sudo ceph -n mon. --fsid [FSID] mon force-create
```

---

## Manager Issues

### Manager Module Failed

```bash
# Check manager status
sudo ceph mgr stat

# Check dashboard
sudo ceph mgr module ls

# Disable problematic module
sudo ceph mgr module disable problematic_module

# Restart manager
sudo ceph orch daemon restart mgr.ceph-ssd.a
```

---

## Performance Issues

### Cluster Slow

**Diagnosis:**

```bash
# Check I/O stats
sudo ceph osd perf

# Check recovery
sudo ceph pg stat

# Check slow requests
sudo ceph log last warn

# Check load on OSDs
top  # or `htop`

# Check disk I/O
iostat -x 1
```

**Solutions:**

1. **If recovery slow:** Increase recovery threads
   ```bash
   sudo ceph config set osd.0 osd_max_backfills 4
   ```

2. **If CPU-bound:** Reduce threads
   ```bash
   sudo ceph config set osd.0 osd_op_num_threads 4
   ```

3. **If disk I/O limited:** Reduce PG split/merge activity
   ```bash
   sudo ceph config set global mon_osd_report_timeout 300
   ```

### High Latency

```bash
# Check network latency
ping -c 5 ceph-ssd

# Check daemon-to-daemon latency
sudo ceph tell osd.0 perf dump | grep "osd.latency"

# Check client latency
sudo ceph log last debug
```

---

## Network Issues

### Cannot Connect to Cluster

```bash
# Check network connectivity
ping ceph-ssd
nc -zv ceph-ssd 6789  # Monitor port
nc -zv ceph-ssd 6800  # OSD port range

# Check firewall
sudo ufw status
sudo ufw allow 6789
sudo ufw allow 6800:7300/tcp
```

### Slow Network

```bash
# Check network stats
iftop
nethogs

# Check MTU size
ip link show
# If not 1500, may need adjustment for performance
```

---

## Storage Issues

### Time Synchronization Issues

**Ceph requires synchronized clocks!**

```bash
# Check time sync
chronyc tracking

# Manually sync if needed
sudo chronyc makestep

# Check clock difference
timedatectl
```

### Disk Health

```bash
# Check disk status
lsblk
sudo smartctl -H /dev/vdb  # Requires smartmontools

# Check disk errors
sudo dmesg | grep -i "I/O error"

# Monitor disk temperature
sudo sensors  # If lm-sensors installed
```

---

## Log Files

### Where to Find Logs

```bash
# Via cephadm (recommended)
sudo ceph orch logs [daemon-name]

# In container volumes
sudo docker logs ceph-mon.ceph-ssd.xx

# Check log directory
ls -la /var/log/ceph/

# System logs
sudo journalctl -u ceph* -n 50
```

### Enable Debug Logging

```bash
# For specific OSD
sudo ceph config set osd.0 debug_osd 20

# For all OSDs
sudo ceph config set global debug_osd 20

# View logs
sudo ceph orch logs osd.0 | tail -f
```

---

## If Everything Fails

### Reset and Start Over (Destructive!)

⚠️ **WARNING:** This erases all data!

```bash
# Stop the cluster
sudo cephadm rm-cluster --fsid [FSID] --force

# Clean up disks
sudo wipefs -af /dev/vdb

# Remove Ceph files
sudo rm -rf /var/lib/ceph/
sudo rm -rf /etc/ceph/

# Start fresh
sudo cephadm bootstrap --mon-ip 192.168.16.51 --single-host-defaults
```

### Get Help

- Check [Official Ceph Documentation](https://docs.ceph.com/en/latest/)
- Search [Ceph Community Forum](https://forum.ceph.io/)
- Check GitHub Issues
- Review `ceph health detail` output

---

## Quick Diagnostic

Run this to get full cluster status:

```bash
#!/bin/bash

echo "=== Cluster Status ==="
sudo ceph -s

echo -e "\n=== Health ==="
sudo ceph health detail

echo -e "\n=== OSD Tree ==="
sudo ceph osd tree

echo -e "\n=== Daemon Status ==="
sudo ceph orch ps

echo -e "\n=== Disk Usage ==="
sudo ceph df

echo -e "\n=== PG Status ==="
sudo ceph pg stat

echo -e "\n=== Version Info ==="
sudo ceph versions
```

Save as `diagnostic.sh`, run with `bash diagnostic.sh`, and share output when reporting issues.

---

**Still stuck?** Check the phase guides or create an issue with your diagnostic output!
