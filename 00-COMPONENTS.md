# Ceph Components Overview

This document explains the key components that make up a Ceph cluster.

## Core Ceph Daemons

### Monitors (ceph-mon)

A [Ceph Monitor](https://docs.ceph.com/en/latest/glossary/#term-Ceph-Monitor) (`ceph-mon`) maintains maps of the cluster state, including:

- Monitor map
- Manager map
- OSD map
- MDS map
- CRUSH map

These maps are **critical cluster state** required for Ceph daemons to coordinate with each other. Monitors are also responsible for managing authentication between daemons and clients.

**Best Practice:** At least three monitors are normally required for redundancy and high availability. For single-node labs, one monitor is acceptable.

---

### Managers (ceph-mgr)

A [Ceph Manager](https://docs.ceph.com/en/latest/glossary/#term-Ceph-Manager) daemon (`ceph-mgr`) is responsible for:

- Keeping track of runtime metrics
- Monitoring the current state of the Ceph cluster
- Tracking storage utilization and performance metrics
- System load monitoring

The Ceph Manager daemons also host **python-based modules** to manage and expose Ceph cluster information, including:

- A web-based [Ceph Dashboard](https://docs.ceph.com/en/latest/mgr/dashboard/#mgr-dashboard) for visual cluster management
- REST API for programmatic access

**Best Practice:** At least two managers are normally required for high availability.

---

### Ceph OSDs (Object Storage Daemons)

An Object Storage Daemon ([Ceph OSD](https://docs.ceph.com/en/latest/glossary/#term-Ceph-OSD), `ceph-osd`) is the workhorse of Ceph. Each OSD:

- Stores data on disk (using **BlueStore** by default)
- Handles data replication and recovery
- Handles data rebalancing
- Provides monitoring information to Ceph Monitors and Managers
- Checks other Ceph OSD Daemons for heartbeats

**Best Practice:** At least three Ceph OSDs are normally required for redundancy and high availability. For single-node labs, one OSD is acceptable.

---

### Metadata Servers (MDS)

A [Ceph Metadata Server](https://docs.ceph.com/en/latest/glossary/#term-Ceph-Metadata-Server) (MDS, `ceph-mds`) is used **only when running CephFS**. It:

- Stores metadata for the [Ceph File System](https://docs.ceph.com/en/latest/glossary/#term-Ceph-File-System)
- Allows CephFS users to run basic commands (`ls`, `find`, etc.)
- Prevents placing a burden on the Ceph Storage Cluster itself

---

### Ceph Object Gateway (RGW)

A [Ceph Object Gateway](https://docs.ceph.com/en/latest/glossary/#term-Ceph-Object-Gateway) (RGW, `ceph-radosgw`) daemon provides:

- A **RESTful gateway** between applications and Ceph storage clusters
- **S3-compatible API** (most commonly used)
- **Swift API** support (alternative)

Think of it as your own private Amazon S3 — applications can use standard S3 clients and SDKs to interact with Ceph.

---

## Ceph Storage Types

### RADOS (Reliable Autonomic Distributed Object Storage)

The foundation of Ceph. RADOS handles:

- Object storage at the lowest level
- Automatic data replication and distribution
- Self-healing and rebalancing

---

### RBD (RADOS Block Device)

Provides **virtual block devices** backed by Ceph:

- Think of it as a virtual hard disk (like AWS EBS volumes)
- Create an "image" and map it to your system as `/dev/rbdX`
- Then format and mount like any regular disk
- Used by OpenStack (Cinder), Kubernetes (CSI), and Proxmox for VM disks

**Key Features:**
- Snapshots and clones
- Thin provisioning
- Live migration

---

### CephFS

A **POSIX-compliant distributed file system** built on top of Ceph:

- Mount it like any regular filesystem (`mount -t ceph`)
- Uses special daemon called **MDS (Metadata Server)** to handle file names, directories, permissions
- Data goes to OSD pools, metadata goes to a separate pool
- Think of it as your own NFS/NAS backed by Ceph

**Key Features:**
- Distributed across multiple OSDs
- Shared filesystem (multiple clients can mount and access simultaneously)
- Automatic rebalancing

---

### RGW (RADOS Gateway)

Provides **S3 and Swift compatible APIs**:

- HTTP/HTTPS access
- Standard S3 clients and SDKs
- Behind the scenes, it stores objects in Ceph pools automatically

---

## Cluster Architecture Concepts

### Placement Groups (PGs)

- Intermediate layer between objects and OSDs
- Ceph maps objects into PGs, then PGs into OSDs
- PGs enable load balancing, replication, and recovery

### CRUSH Map

- **Controlled Replication Under Scalable Hashing**
- Deterministic algorithm for where to place data
- Considers cluster topology (racks, hosts, etc.)
- For single-host deployments: `osd_crush_chooseleaf_type = 0` (allows replicas on same host)

### Replication

- **Default:** Each object is replicated to multiple OSDs
- **Replica size:** How many copies (typically 3 for production, 1-2 for labs)
- **Min size:** Minimum replicas before writes are successful

---

## Dashboard & Monitoring

After bootstrap, you'll have access to:

- **Ceph Dashboard** — web UI for cluster management
- **Prometheus metrics** — for monitoring and alerting
- **CLI tools** — `ceph -s`, `ceph df`, etc.

---

**Next:** [System Preparation](./01-SYSTEM-PREPARATION.md)
