# Docker Issues and Tips (aufs/overlay/btrfs..)

Picked up and categorized subjectively. Comments and pull requests are welcome.

## Storage Drivers
### AUFS
|Issue|Abstract|Impact|Reproducibility|Cause|Solution|Notes|
|---|---|---|---|---|---|---|
|:white_check_mark: [#783](https://github.com/docker/docker/issues/783)|Cannot access to a directory due to a permission error|:neutral_face: Medium|:smiley: Easy|Expected AUFS behavior. `dirperm1` mount option fixes this issue. |[Update the kernel (AUFS >= 2008xxxx?) and Docker daemon (>= 1.7)](https://github.com/docker/docker/issues/783#issuecomment-146455363)|Confirm: `docker info | grep "Dirperm1 Supported: true"`|
|:white_check_mark: [#18180](https://github.com/docker/docker/issues/18180)|A process becomes a zombie and hangs up |:scream: High|:scream: Hard(multiprocessor)<br> :smiley: Easy(uniprocessor)|Compatibility between the kernel and AUFS|[Update the kernel (AUFS >= 20160111)](https://github.com/docker/docker/issues/18180#issuecomment-187583209)|Especially Java apps and MongoDB are affected|
|:white_large_square: [#20199](https://github.com/docker/docker/issues/20199)|`fcntl(F_SETFL, O_APPEND)` is ignored and hence data can be corrupted|:scream: High|:smiley: Easy|AUFS bug|None (Workaround: [patch](https://github.com/docker/docker/issues/20199#issuecomment-182944585))|Especially Dovecot is affected|
|:white_large_square: [#20240](https://github.com/docker/docker/issues/20240)|Weird permission even though `dirperm1` is enabled|:neutral_face: Medium|:scream: Hard|Unanalyzed|None||

Non-bug issues:
* AUFS is not available in the mainline kernel．Only a few distros (Ubuntu, Boot2Docker, ..) support AUFS, but even for Ubuntu, Canonical says ["AUFS will disappear"](https://lists.ubuntu.com/archives/ubuntu-devel/2012-March/034880.html).

### Overlay
|Issue|Abstract|Impact|Reproducibility|Cause|Solution|Notes|
|---|---|---|---|---|---|---|
|:white_large_square: [#10180](https://github.com/docker/docker/issues/10180)|File corruption|:scream: High|:scream: Hard|Unanalyzed|None|Especially RPMs are affected|
|:white_large_square: [#12080](https://github.com/docker/docker/issues/12080)|Cannot use UNIX domain sockets|:neutral_face: Medium|:smiley: Easy|Overlay Bug|None (Workaround: [patch](https://github.com/docker/docker/issues/12080#issuecomment-146946771))||
|:white_large_square: [#19082](https://github.com/docker/docker/issues/19082)|Weird behavior after removing the current directory|:smiley: Low|:smiley: Easy|Overlay Bug|None (Workaround: [patch](https://lkml.org/lkml/2016/1/8/482))||
|:white_large_square: [#19647](https://github.com/docker/docker/issues/19647)|Untar fails intermittently|:scream: High|:scream: Hard|Unanalyzed (Overlay bug related to symbolic links?)|None||
|:white_large_square: [#19758](https://github.com/docker/docker/issues/19758)|Daemon hangs up after frequent `docker run`|:scream: High|:scream: Hard|Unanalyzed (Overlay bug related to the number of processors?)|None||

Non-bug issues:
* :scream: High inode usage
* Red Hat says OverlayFS is [Tech Preview](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/7.2_Release_Notes/technology-preview-file_systems.html)

### BtrFS
|Issue|Abstract|Impact|Reproducibility|Cause|Solution|Notes|
|---|---|---|---|---|---|---|
|:white_check_mark: [#19073](https://github.com/docker/docker/issues/19073)|`sendfile(2)` can be unkillable|:smiley: Low|:smiley: Easy|BtrFS bug|None|Not likely to happen in production, but needs consideration for public PaaS|
|:white_large_square: [#20080](https://github.com/docker/docker/issues/20080)|cgroups kmem limit leads crash and data corruption|:scream: High|:smiley: Easy?|Btrfs bug|Avoid kmem limit configuration?||

Non-bug issues:

 * Slow [#10161](https://github.com/docker/docker/issues/10161)
 * No page sharing (e.g. same DLLs are loaded redundantly) http://comments.gmane.org/gmane.comp.sysutils.docker.devel/1384
 * Docker says BtrFS is [Experimental](https://docs.docker.com/engine/userguide/storagedriver/btrfs-driver/). Red Hat says BtrFS is [Tech Preview](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/7.2_Release_Notes/technology-preview-file_systems.html).

### ZFS

|Issue|Abstract|Impact|Reproducibility|Cause|Solution|Notes|
|---|---|---|---|---|---|---|
|:white_check_mark: [#20153](https://github.com/docker/docker/issues/20153)|Some operations fail due to `EBUSY`|:neutral_face: Medium|:neutral_face: Medium|Daemon bug|[Update Docker daemon](https://github.com/docker/docker/commit/803e3d4d1e7b9f029bbe31a80197403bf4d27252)||

Non-bug issues:

 * Docker says ZFS is [not recommended for production](https://docs.docker.com/engine/userguide/storagedriver/zfs-driver/).

### DeviceMapper

|Issue|Abstract|Impact|Reproducibility|Cause|Solution|Notes|
|---|---|---|---|---|---|---|
|:white_check_mark: [#4036](https://github.com/docker/docker/issues/4036)|Mount fails|:scream: High|:smiley: Easy|udev sync disabled|Use a Docker daemon binary which supports udev sync|Confirm: `docker info | grep "Udev Sync Supported: true"`|
|:white_large_square: [#5684](https://github.com/docker/docker/issues/5684)|Cannot restart containers after restaring the daemon|:scream: High|:smiley: Easy|Unanalyzed|None||
|:white_large_square: [#20401](https://github.com/docker/docker/issues/20401)|Infinite “mount/remount” loop, which makes the system unresponsive|:scream: High|:scream: High|Unanalyzed (perhaps related to XFS)|None||

Non-bug issues:

 * Slow [#10161](https://github.com/docker/docker/issues/10161)
 * No page sharing (e.g. same DLLs are loaded redundantly) http://comments.gmane.org/gmane.comp.sysutils.docker.devel/1384

### So which storage driver should I use?

It totally depends on your workload, but Docker, Inc. says AUFS and Devicemapper (direct-lvm) are "production-ready".

https://github.com/docker/docker/blob/master/docs/userguide/storagedriver/selectadriver.md#future-proofing

![driver-pros-and-cons.png](https://raw.githubusercontent.com/docker/docker/89923c1f32aeff5bf11fbb04723dd0e154559545/docs/userguide/storagedriver/images/driver-pros-cons.png)

Although not listed in the above table, VFS driver is also attractive for its robustness.

Links:

 * https://jpetazzo.github.io/assets/2015-03-03-not-so-deep-dive-into-docker-storage-drivers.html#1
 * http://www.projectatomic.io/docs/filesystems/


#### Anyway...
You know, containers should be "immutable" and "disposable".

For persistent data and some special temporary data, you should better consider using an external volume (`docker run -v`).

Links:

* https://github.com/emccode/rexray


## Network
|Issue|Abstract|Impact|Reproducibility|Cause|Solution|Notes|
|---|---|---|---|---|---|---|
|:white_large_square: [#18776](https://github.com/docker/docker/issues/18776)|TCP checksums are ignored|:scream: High|:scream: Hard|Kernel bug|[Update the kernel](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/?id=ce8c839b74e3017996fad4e1b7ba2e2625ede82f)|[blog](https://medium.com/vijay-pandurangan/linux-kernel-bug-delivers-corrupt-tcp-ip-data-to-mesos-kubernetes-docker-containers-4986f88f7a19)|
