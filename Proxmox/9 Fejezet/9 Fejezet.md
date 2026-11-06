## 9 Fejezet Network and MTU Testing

**Mesh network:** For small clusters with 3-5 nodes, if a 10+ Gbps switch is not available, directly connecting the physical network cards (**Mesh network**) is an officially supported option for Ceph.

**Ping test with Jumbo Frames:** When changing networks, the correct operation of the MTU (e.g., 9000) must be tested by calculating the IP+ICMP overhead. For a 9000 MTU, the packet size should be **8972 bytes** for IPv4 (`ping -M do -s 8972 <IP>`), and **8952 bytes** for IPv6.

**IPv6 settings:** If you switch from IPv4 to IPv6 (or vice versa), the `ms_bind_ipv4` and `ms_bind_ipv6` variables must be changed to `true` or `false` in the `ceph.conf` file.

## 2. OSD Creation (DB/WAL) and Filestore

**DB/WAL sizing fallback logic:** If you add an NVMe/SSD cache to an HDD, but do not specify the `-db_size` or `-wal_size` dimensions, Ceph will look for the sizes in the following order:

1. `ceph.conf` OSD section,
2. `ceph.conf` Global section,
3. As a default setting, **10%** of the disk size for the DB, and **1%** for the WAL.

**Filestore:** Although Bluestore is the default since Nautilus, the old Filestore-based OSD can still be created, but no longer with the `pveceph` command, but directly with the backend tool:

```bash
ceph-volume lvm create --filestore --data /dev/sd[X] --journal /dev/sd[Y]
```

## 3. Encrypted OSD Lockbox Keys

> **Danger:** Every encrypted OSD uses a dedicated `client.osd-lockbox.<uuid>` user to retrieve the disk encryption key. This key must **never** be rotated with the normal `ceph auth` command, because the LVM tag of the disk and the Ceph database must be updated simultaneously. Only the built-in migration tool of PVE is suitable for this.

## 4. Ceph Configuration Hierarchy

**Override:** If you set a value in the MON database with the `ceph config set` command, but the option also exists in the local `ceph.conf` file, the **local file overrides the central one** (the MON DB has a lower priority).

## 5. Pool and Erasure Coding Extra Details

**Pool options:** During GUI/CLI creation, extras can also be specified, such as:

- **Target Ratio** (expected ratio of data compared to other pools),
- **Target Size** (expected size in gigabytes),
- **Min. # of PGs** (the autoscaler cannot merge PGs below this value).

**EC Pool math:** The `min_size` value of Erasure Coded pools is determined as follows:

- if $m=1$, then `min_size` equals $k$,
- if $m>1$, then $min\_size = k+1$.

(The documentation officially recommends $k+2$ for security reasons).

**Attaching an EC Pool as PVE Storage:** Since a mandatory metadata pool is also created alongside the EC pool, the `--data-pool` switch must be used when adding it to the Proxmox storage:

```bash
pvesm add rbd <name> --pool <meta_pool> --data-pool <data_pool>
```

## 6. PG Autoscaler and CRUSH Relationships

**Autoscaler limit:** The PG autoscaler intervenes by default only if the actual PG count deviates by a factor of **3** from the optimal. Since this intervention involves massive data movement, it is advisable to set the **Target Size** in advance.

**CRUSH incompatibility:** The autoscaler can only scale the pools if all pools use the same type of CRUSH rule (e.g., either a **Device Class** is specified for all of them, or for none). In the case of mixed rules, the autoscaler skips the faulty pools.

## 7. Deeper CephFS Options

**Multiple Active MDS:** Starting from the Luminous version, it is possible to run multiple active Metadata servers simultaneously. This is only necessary for an extreme number of clients reading/writing in parallel, as the MDS is rarely a bottleneck.

**CephFS structure:** The `pveceph fs create` command creates two pools in the background:

- `cephfs_data` pool (with 128 PGs by default),
- `cephfs_metadata` pool (where the number of PGs is always a quarter of the data pool, in this case 32).

**Deletion switch:** When deleting the file system, the `--remove-storages` switch can be used to also remove the entry from `storage.cfg`.

## 8. Maintenance and Troubleshooting Extras

**Permanently muting a health alert:** By using the `--sticky` (**Keep Mute** on GUI) switch with the `ceph health mute <name>` command, the alert remains muted even if the problem worsens (e.g., another OSD fails with the same error).

**Basic logs:** For deeper diagnostics, the `/var/log/ceph/` directory and the system log are required:

```bash
journalctl --since "2 days ago"
```

## 9. Full Process of Key Rotation (CVE-2025-30156)

The previous note only mentioned the rotation of cluster-keys, but the migration of user and storage keys is a much more complex, multi-step process:

1. **Check client compatibility:** CephFS and LXC RBD mounts use the Linux kernel client. For `aes256k` encryption, at least a **7.0** Linux kernel is mandatory on the server!
2. **Temporary key staging (Staging):** The updating of storage keys must be initiated with the `--rotate-all-storage-keys --rotate-admin-key` switches. This "stages" the new key, but leaves the old one valid until the clients are updated.
3. **Re-reading clients:** Running VMs must be live migrated, or completely stopped and started. A simple restart (`guest reboot`) is **not enough** to re-read the key! Containers must be stopped and then restarted.
4. **Confirmation or Rollback:** Once all clients have switched, the process can be finalized with the `--confirm-all-clients-refreshed` switch. If there is a problem, the new key can be revoked with the `--abort-staged-key` command.
5. **Emergency Override:** If you lock yourself out of the system, you can regain access with the old keys by temporarily setting `mon_auth_emergency_allowed_ciphers` in `ceph.conf`.
