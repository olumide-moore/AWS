# EBS Volumes Setup with EC2 Instance

## Overview
- **EBS (Elastic Block Store)** provides:
  - **Volumes**: Virtual hard disks for EC2 instances (block storage).
  - **Snapshots**: Backups of volumes.
- Volumes must be in the **same Availability Zone** as the EC2 instance.
- EBS automatically replicates data within the same AZ.

## EBS Volume Types
| Type | Use Case | Notes |
|:---|:---|:---|
| General Purpose (gp2/gp3) | Web servers, general workloads | Best default option |
| Provisioned IOPS | High-performance DBs | High IOPS, expensive |
| Throughput Optimized HDD | Big data, log processing | High throughput |
| Cold HDD | Infrequently accessed data | Low cost |
| Magnetic | Archives, backups | Very cheap, slow |

## Practical Setup
1. **Launch EC2 Instance** (CentOS 9, t2.micro, 10GB gp2).
2. **SSH Access**: Update security group to your IP, connect using key.
3. **Install Web Server**: Deploy via user data script or manually.

## Adding a New Volume
1. Create a **5GB gp2** volume in the **same AZ**.
2. **Attach** to the instance.
3. **Partition**:
   ```bash
   fdisk /dev/xvdf
   # n → p → 1 → Enter → Enter → w
   fdisk -l #to view list of disks and partitions
   ```
4. **Format**:
   ```bash
   mkfs #(and tab twice to view formatting options)
   mkfs.ext4 /dev/xvdf1
   ```
5. **Mount**:
   ```bash
   mkdir /var/www/html/images
   mount /dev/xvdf1 /var/www/html/images
   df -h #to view mounted partition
   umount /var/www/html/images # to unmount
   ```
6. **Permanent Mount**:
   - Edit `/etc/fstab`:
     ```
     /dev/xvdf1  /var/www/html/images  ext4  defaults  0  0
     ```
   - Apply:
     ```bash
     mount -a
     ```
7. **Restore Data** and **restart web service**:
   ```bash
   mv /tmp/img-bakups/* /var/www/html/images/
   systemctl restart httpd
   ```

 
# **Snapshots**

## **Session Overview**
- Continuation from previous section.
- Rename EC2 instance from `web01` to `db01`.
- If you terminated your instance, relaunch a new **CentOS7** EC2 instance.

---

## **Unmounting a Volume**

1. **Check mounted volumes**:
   ```bash
   df -h
   ```

2. **Unmount a volume**:
   ```bash
   umount /var/www/html/images
   ```

3. **If unmount fails** (due to running processes like Apache):
   - Install `lsof`:
     ```bash
     yum install lsof -y
     ```
   - Find processes using the mount:
     ```bash
     lsof /var/www/html/images
     ```
   - Kill the process (only if needed):
     ```bash
     kill <PID>
     ```
   - **Avoid** force unmounting (`umount -l`) unless necessary.

---

## **Detaching and Deleting a Volume**

- Go to AWS Console → **Volumes**.
- **Detach Volume** cleanly.
- If needed, force detach or shutdown instance.
- **Delete unused volumes** to avoid unwanted charges.

---

## **Creating a New Volume for Database (MySQL)**

1. **Create new volume** (5GB, General Purpose, correct AZ).
2. **Attach** it to the instance.
3. **Format** the volume:
   ```bash
   fdisk /dev/xvdf
   ```
   - Create a new primary partition (`n` → `p` → `1`).
   - Allocate 3GB (`+3G`).
   - Write changes (`w`).
4. **Create filesystem**:
   ```bash
   mkfs.ext4 /dev/xvdf1
   ```

5. **Mount the filesystem**:
   - Create MySQL directory:
     ```bash
     mkdir -p /var/lib/mysql
     ```
   - Update `/etc/fstab` for persistent mounting.
   - Mount all:
     ```bash
     mount -a
     ```

---

## **Installing and Configuring MariaDB**

- Install MariaDB:
  ```bash
  yum install mariadb-server -y
  ```
- Start service:
  ```bash
  systemctl start mariadb
  ```
- Check status:
  ```bash
  systemctl status mariadb
  ```

> **Note**: Disable SELinux if there are service issues.

---

## **Taking Snapshots**

- Go to **Volumes** → select volume → **Create Snapshot**.
- Add **description and tags**.
- Wait for snapshot to complete (status: `completed`).

---

## **Simulating Data Loss and Recovery**

1. **Simulate data loss**:
   ```bash
   cd /var/lib/mysql
   rm -rf *
   ```

2. **Stop MariaDB service**:
   ```bash
   systemctl stop mariadb
   ```

3. **Unmount and detach** corrupted volume.
   - Rename volume to something like "corrupted" for easy tracking.

4. **Create new volume from snapshot**:
   - Go to **Snapshots** → Create Volume.
   - You can change:
     - Volume type
     - Volume size
     - Availability zone
     - Encryption status

5. **Attach new volume** to instance.

6. **Mount new volume**:
   ```bash
   mount -a
   ```

7. **Verify data recovery** at `/var/lib/mysql`.

---

## **Extra Use Cases for Snapshots**

- **Change Volume Type** (e.g., General Purpose → Provisioned IOPS).
- **Increase Volume Size**.
- **Move Volumes Across Availability Zones**.
- **Encrypt an Unencrypted Volume**.
- **Create AMI** from a root volume snapshot.
- **Copy Snapshot** to another region.
- **Share Snapshot** with other AWS accounts.

---

## **Clean Up**

1. **Terminate EC2 instance**.
2. **Delete attached volumes**.
3. **Delete snapshots**.
4. Verify:
   - **Instances** = 0 (or only terminated ones).
   - **Volumes** = 0.
   - **Snapshots** = 0.

---

