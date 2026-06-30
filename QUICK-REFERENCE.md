# Ceph Quick Reference

Handy commands for daily Ceph operations.

## Cluster Status & Health

```bash
# Overall cluster status
sudo ceph -s

# Detailed health information
sudo ceph health detail

# Monitor map
sudo ceph mon map

# OSD map
sudo ceph osd map pool-name object-name
```

## Monitor Commands

```bash
# Monitor status
sudo ceph mon stat

# List monitors
sudo ceph mon ls

# Get monitor quorum
sudo ceph quorum_status
```

## OSD Commands

```bash
# List all OSDs
sudo ceph osd ls

# OSD tree (hierarchy)
sudo ceph osd tree

# OSD status
sudo ceph osd status

# Mark OSD down/out (maintenance)
sudo ceph osd down 0
sudo ceph osd out 0

# Bring OSD back up/in
sudo ceph osd in 0
sudo ceph osd up 0

# Reweight an OSD
sudo ceph osd reweight 0 0.75

# Crush map info
sudo ceph osd crush show-tree
```

## Pool Commands

```bash
# List pools
sudo ceph osd pool ls

# Pool stats
sudo ceph osd pool stats

# Create pool
sudo ceph osd pool create my-pool 32

# Delete pool (requires confirmation)
sudo ceph osd pool delete my-pool my-pool --yes-i-really-really-mean-it

# Modify pool
sudo ceph osd pool set my-pool size 3
sudo ceph osd pool set my-pool min_size 2

# Pool quota
sudo ceph osd pool set-quota my-pool max_bytes 1099511627776
```

## RGW Commands

```bash
# List realms
sudo radosgw-admin realm list

# List zonegroups
sudo radosgw-admin zonegroup list

# List zones
sudo radosgw-admin zone list

# Create user
sudo radosgw-admin user create --uid=myuser --display-name="My User"

# List users
sudo radosgw-admin user list

# Get user info
sudo radosgw-admin user info --uid=myuser

# Delete user
sudo radosgw-admin user remove --uid=myuser

# Create access key
sudo radosgw-admin user modify --uid=myuser --access-key --secret-key

# List buckets
sudo radosgw-admin bucket list

# Get bucket info
sudo radosgw-admin bucket stats --bucket=my-bucket

# Delete bucket
sudo radosgw-admin bucket rm --bucket=my-bucket
```

## CephFS Commands

```bash
# Filesystem status
sudo ceph fs status

# List filesystems
sudo ceph fs ls

# Filesystem dump
sudo ceph fs dump

# MDS status
sudo ceph mds stat

# Remove filesystem
sudo ceph fs rm cephfs --yes-i-really-mean-it

# Mount CephFS
sudo mount -t ceph ceph-ssd:/ /mnt/cephfs -o name=admin,secret=$(sudo ceph auth get-key client.admin)

# Unmount CephFS
sudo umount /mnt/cephfs
```

## RBD Commands

```bash
# List RBD pools
sudo rbd pool ls

# List images in pool
sudo rbd ls -p rbd-pool

# Create image
sudo rbd create rbd-pool/my-image --size 10G

# Image info
sudo rbd info rbd-pool/my-image

# List mapped images
sudo rbd showmapped

# Map image
sudo rbd map rbd-pool/my-image

# Unmap image
sudo rbd unmap /dev/rbd0

# Delete image
sudo rbd rm rbd-pool/my-image

# Create snapshot
sudo rbd snap create rbd-pool/my-image@snapshot1

# List snapshots
sudo rbd snap ls rbd-pool/my-image

# Delete snapshot
sudo rbd snap rm rbd-pool/my-image@snapshot1

# Clone from snapshot
sudo rbd clone rbd-pool/my-image@snapshot1 rbd-pool/my-image-clone

# Resize image
sudo rbd resize rbd-pool/my-image --size 20G

# Image rename
sudo rbd mv rbd-pool/old-name rbd-pool/new-name
```

## Authentication & Keys

```bash
# List authentication
sudo ceph auth list

# Get key for user
sudo ceph auth get-key client.admin

# Create user
sudo ceph auth add client.myuser mon 'allow r' osd 'allow rw pool=my-pool'

# Delete user
sudo ceph auth del client.myuser

# Export keys
sudo ceph auth export > ceph.keys

# Import keys
sudo ceph auth import -i ceph.keys
```

## Orchestration (cephadm)

```bash
# List services
sudo ceph orch ls

# List running daemons
sudo ceph orch ps

# Filter by service
sudo ceph orch ps --service-name=osd

# View daemon logs
sudo ceph orch logs mon.ceph-ssd

# Restart daemon
sudo ceph orch daemon restart mon.ceph-ssd

# Stop daemon
sudo ceph orch daemon stop mon.ceph-ssd

# Start daemon
sudo ceph orch daemon start mon.ceph-ssd

# List devices
sudo ceph orch device ls

# Add OSD
sudo ceph orch daemon add osd ceph-ssd:/dev/vdb

# Apply service
sudo ceph orch apply rgw default default --placement="1 ceph-ssd" --port=8080

# Remove service
sudo ceph orch rm rgw.default
```

## Ceph Shell

```bash
# Enter cephadm shell
sudo cephadm shell

# Inside shell, run commands without sudo:
ceph -s
ceph osd tree
```

## Configuration

```bash
# List all configuration
sudo ceph config dump

# Get specific config value
sudo ceph config get global osd_pool_default_size

# Set configuration
sudo ceph config set global osd_pool_default_size 2

# Get config from file
cat /etc/ceph/ceph.conf
```

## Troubleshooting

```bash
# Cluster health (verbose)
sudo ceph health detail

# Check if all OSDs are up
sudo ceph osd stat

# Check placement groups status
sudo ceph pg stat

# Check slow requests
sudo ceph log last

# View recent events
sudo ceph log last warn

# Check daemon versions
sudo ceph versions

# Check cluster usage
sudo ceph df
```

## Dashboard

```bash
# Enable dashboard
sudo ceph mgr module enable dashboard

# Disable dashboard
sudo ceph mgr module disable dashboard

# Get dashboard URL
sudo ceph mgr services

# Reset dashboard password
sudo ceph dashboard ac-user-set-password admin newpassword

# Set dashboard SSL certificate
sudo ceph dashboard set-ssl-certificate -i /path/to/cert.crt
```

## Performance Tuning

```bash
# Enable autotune
sudo ceph config set global osd_memory_target_autotune true

# Set memory target (MB)
sudo ceph config set osd.0 osd_memory_target 1073741824

# Monitor I/O performance
sudo ceph osd perf

# Check recovery progress
sudo ceph pg stat
```

## Backup & Restore

```bash
# Export cluster configuration
sudo ceph config dump > cluster.conf

# Export auth keys
sudo ceph auth export > ceph.keys

# Export OSD map
sudo ceph osd getmap > osdmap.bin

# Export monitor map
sudo ceph mon getmap > monmap.bin

# View exported map
sudo osdmaptool --print osdmap.bin
```

## Useful Information Queries

```bash
# Full cluster map
sudo ceph mon dump

# OSD dump
sudo ceph osd dump

# MDS dump
sudo ceph mds dump

# PG map
sudo ceph pg dump

# Pool detailed stats
sudo ceph osd pool get-quota
```

## Emergency Commands

```bash
# ⚠️  Force start cluster (dangerous!)
# sudo ceph -n mon. --fsid de288192-0c9c-11f1-b95c-020100d7008e mon force-create

# ⚠️  Rebuild OSD filesystem
# sudo ceph-volume lvm destroy --physical-device /dev/vdb
# sudo ceph orch daemon add osd ceph-ssd:/dev/vdb

# ⚠️  Reinstall all packages
# sudo cephadm install

# Check which OSDs are down
sudo ceph osd find <id>

# Check placement group status
sudo ceph pg <pgid> query
```

---

**Tip:** Most commands can be run via `cephadm shell` without requiring `sudo`:

```bash
sudo cephadm shell
# Inside the shell:
ceph -s
ceph osd tree
```

For detailed information on any command:

```bash
sudo ceph [command] --help
```

---

**Need more help?**
- See the full guides in `/docs/`
- Check [Official Ceph Documentation](https://docs.ceph.com/en/latest/)
