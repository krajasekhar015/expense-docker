# Resize EBS Storage

**Check the partitions**
```
lsblk
```
`lsblk` (list block devices) is a command in Linux that provides detailed information about all available block devices, such as hard drives, SSDs, and their partitions. It presents this information in a hierarchical format, making it easier to understand the relationships between devices and their partitions.

**Re-size partitions**
```
sudo growpart /dev/nvme0n1 4
```
Here,
* `growpart` is a command-line utility that is part of the `cloud-guest-utils` package. It is specifically designed to resize existing partition to fill the available space
* `/dev/nvme0n1` specifies the disk where the partition you want to resize is located. `nvme0n1` indicates the first NVMe disk. NVMe (Non-Volatile Memory Express) is a high-speed interface used for SSDs
* `4` specifies the partition number you want to resize. In this case, it is the fourth partition on the specified disk to use the remaining unallocated space on the disk.

**Extend the Logical Volumes** <br>
Decide how much space to allocate to each logical volume. For example, to extend both the root and /var logical volumes:

```
sudo lvextend -l +50%FREE /dev/RootVG/rootVol
sudo lvextend -l +50%FREE /dev/RootVG/varVol
```
Here,
* `lvextend` is the command used to extend the size of a logical volume
* `-l +50%FREE`: Here `-l` option specifies the size to which the logical volume should be extended in terms of logical extents and `+50%FREE` argument means that the logical volume should be extended by 50% of the free space available in the volume group
* `/dev/RootVG/rootVol` specifies the path to the logical volume that you want to extend. In this case, `RootVG` is the name of the volume group, and `rootVol` is the name of the logical volume

**Re-size the filesystems** <br>
After extending the logical volumes, resize the filesystems to utilize the additional space.

For the **root filesystem**:
```
sudo xfs_growfs /
```
For the **/var filesystem**:
```
sudo xfs_growfs /var
```
Here,
* `xfs_growfs` is the command specifically designed to grow (extend) an XFS filesystem. XFS is a high-performance filesystem commonly used in enterprise environments
* `/` and `/var`: This indicates the mount point of the filesystem you want to extend
    * In this case, `/` refers to the root filesystem and `/var` is usually a directory that contains variable data such as logs, databases, and temporary files