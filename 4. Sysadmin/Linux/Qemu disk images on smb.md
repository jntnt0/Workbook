# Accessing QEMU/KVM Disk Images Stored on SMB Shares

## Overview

Running QEMU/KVM virtual machines directly from disk images stored on SMB network shares requires proper mount configuration. Mounting via GVFS (the GNOME Virtual File System) — which happens automatically when browsing network shares through a file manager — does not provide the permissions or compatibility that QEMU requires. This guide covers the correct approach.

---

## Why GVFS Mounts Fail with QEMU

When a network share is mounted via GVFS, it is accessible under a path like:

```
/run/user/1000/gvfs/smb-share:server=<host>,share=<share>/
```

QEMU cannot reliably open disk images from GVFS mounts because:

- `chmod` and file permission changes are not supported — permissions are governed by the remote SMB server
- GVFS mounts are scoped to the current user session and do not behave like standard POSIX filesystems
- QEMU requires direct, low-level file access that GVFS does not provide

---

## Solution 1 — Mount the Share with CIFS (Recommended)

Replace the GVFS mount with a proper CIFS mount that gives QEMU the access it needs.

**Unmount the existing GVFS mount if present:**

```bash
fusermount -u "/run/user/1000/gvfs/smb-share:server=<host>,share=<share>"
```

**Create a local mount point:**

```bash
sudo mkdir -p /mnt/<share-name>
```

**Mount the share with CIFS:**

```bash
sudo mount -t cifs //<server-ip>/<share-name> /mnt/<share-name> \
    -o username=<smb-username>,password=<smb-password>,uid=$(id -u),gid=$(id -g)
```

**Run QEMU against the mounted image:**

```bash
qemu-system-x86_64 \
    -enable-kvm \
    -m 4096 \
    -cpu host \
    -drive file=/mnt/<share-name>/<path-to-image>.qcow2,format=qcow2 \
    -boot c \
    -net nic \
    -net user
```

**Make the mount persistent across reboots** by adding an entry to `/etc/fstab`:

```
//<server-ip>/<share-name>  /mnt/<share-name>  cifs  username=<user>,password=<pass>,uid=1000,gid=1000,_netdev  0  0
```

> Store credentials in a protected file instead of plaintext in fstab:
> 
> ```
> //<server-ip>/<share-name>  /mnt/<share-name>  cifs  credentials=/etc/samba/<creds-file>,uid=1000,gid=1000,_netdev  0  0
> ```
> 
> Where `/etc/samba/<creds-file>` contains:
> 
> ```
> username=<smb-username>
> password=<smb-password>
> ```
> 
> ```bash
> sudo chmod 600 /etc/samba/<creds-file>
> ```

---

## Solution 2 — Copy the Image Locally

If the share mount is temporary or the image is small enough, copy it locally before running:

```bash
cp /mnt/<share-name>/<path-to-image>.qcow2 ~/images/<image>.qcow2
```

Then run QEMU against the local copy. This eliminates network latency and permission issues entirely.

---

## Solution 3 — Run QEMU as Root (Not Recommended)

If the GVFS mount is the only available path, QEMU can be run with `sudo` to bypass permission restrictions:

```bash
sudo qemu-system-x86_64 \
    -enable-kvm \
    -m 4096 \
    -cpu host \
    -drive file="/run/user/1000/gvfs/smb-share:server=<host>,share=<share>/<path>.qcow2",format=qcow2 \
    -boot c
```

> This is not suitable for regular use. Running QEMU as root expands the attack surface and should only be used for quick one-off testing.

---

## Verifying SMB Share Permissions

If CIFS mounts still fail, verify permissions on the SMB server side:

- The connecting user must have read/write access to the share
- The share must not be configured as read-only
- If using Samba, check `/etc/samba/smb.conf` for `read only = no` and correct `valid users`

**Test connectivity and authentication from the Linux client:**

```bash
smbclient //<server-ip>/<share-name> -U <username>
```

---

## Common Issues

|Symptom|Likely Cause|Fix|
|---|---|---|
|`Permission denied` on GVFS path|GVFS does not support POSIX permissions|Use CIFS mount instead|
|QEMU fails to open image on CIFS mount|Mount missing `uid`/`gid` options|Remount with `uid=$(id -u),gid=$(id -g)`|
|CIFS mount drops after reboot|Not in `/etc/fstab`|Add persistent fstab entry with `_netdev` flag|
|Slow VM performance over network|Network latency on disk I/O|Copy image locally or use NFS instead of CIFS|