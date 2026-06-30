# Ceph Cluster Setup Guide

A complete step-by-step guide for setting up a Ceph storage cluster with all major components: OSD, RGW (S3-compatible storage), CephFS (distributed filesystem), and RBD (block storage).

##  Table of Contents

1. [Ceph Components Overview](./00-COMPONENTS.md)
2. [System Preparation](./01-SYSTEM-PREPARATION.md)
3. [Install cephadm](./02-INSTALL-CEPHADM.md)
4. [Bootstrap the Cluster](./03-BOOTSTRAP-CLUSTER.md)
5. [Verify Installation](./04-VERIFY-INSTALLATION.md)
6. [Add OSD (Object Storage Daemon)](./05-ADD-OSD.md)
7. [Deploy RGW (S3-Compatible Gateway)](./06-DEPLOY-RGW.md)
8. [Deploy CephFS (Distributed Filesystem)](./07-DEPLOY-CEPHFS.md)
9. [Setup RBD (Block Storage)](./08-SETUP-RBD.md)

##  Quick Start

This guide walks through installing Ceph on a single host with:
- 1 Monitor (ceph-mon)
- 2 Managers (ceph-mgr)
- 1 OSD (ceph-osd)
- 1 RGW daemon (S3 gateway)
- 1 MDS daemon (CephFS metadata server)
- RBD block storage

##  Requirements

- Linux system (Ubuntu 22.04 LTS tested)
- Docker installed
- At least one additional disk for OSD storage
- 192.168.16.51 IP address (adjust as needed)

##  How to Use

Each phase is a separate markdown file. Follow them in order:

1. Read the overview in [Ceph Components](./docs/00-COMPONENTS.md)
2. Start with [System Preparation](./docs/01-SYSTEM-PREPARATION.md)
3. Follow each phase sequentially

Each guide includes explanations of what each command does and expected outputs.

##  Key Concepts

- **OSD**: Object Storage Daemon - stores data on disk
- **Monitor**: Maintains cluster state and configuration
- **Manager**: Tracks metrics and hosts the web dashboard
- **RGW**: S3-compatible object storage gateway
- **CephFS**: POSIX-compliant distributed filesystem
- **RBD**: Virtual block devices (like EBS volumes)
- **MDS**: Metadata Server for CephFS

##  Important Notes

- The `--single-host-defaults` flag is used for single-node deployments
- Pool replica size is set to 1 (for single OSD deployments)
- Dashboard credentials are provided after bootstrap
- Kernel CephFS driver is used for mounting

##  References

- [Official Ceph Documentation](https://docs.ceph.com/en/latest/)
- [Ceph Architecture Guide](https://docs.ceph.com/en/latest/architecture/)
- [cephadm Documentation](https://docs.ceph.com/en/latest/cephadm/)

##  Need Help?

- Each guide includes explanations of what happens "behind the scenes"
- Check the official Ceph docs for advanced configuration
- Common commands like `ceph -s` show cluster status

---

**Start with:** [Ceph Components Overview](./docs/00-COMPONENTS.md)
