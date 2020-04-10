The development of this role has been stopped because of the lack of proper ZFS UEFI drivers.

# Motivation

I wanted to use this bootloader because it has UEFI support and touch/mouse support. Also a good theming motor and a simple configuration file.

# Problems found

I also want to use bootenvs making ZFS snapshots so this project was not only a role for installing rEFInd but a plugin fot [zedenv](zedenv.readthedocs.io/) supporting rEFInd.

I tryed to load a ZFS driver and every dataset I created resulted on an error. This is EFI Shell outputs:

```
Device mapping table
  fs0   :HardDisk - Alias hd20b blk0
         pciRoot(0x0)/.*/NVMe(.*)/HD(1,GPT,PARTUUID)
  blk0  :HardDisk - Alias hd20b fs0
         pciRoot(0x0)/.*/NVMe(.*)/HD(1,GPT,PARTUUID)
  blk1  :HardDisk - Alias hd20c
         pciRoot(0x0)/.*/NVMe(.*)/HD(2,GPT,PARTUUID)
  hd20b :HardDisk - Alias fs0 blk0
         pciRoot(0x0)/.*/NVMe(.*)/HD(1,GPT,PARTUUID)
  hd20c :HardDisk - Alias blk1
         pciRoot(0x0)/.*/NVMe(.*)/HD(2,GPT,PARTUUID)

Shell> blk1:

blk1:\> ls
ls/dir: Cannot open current directory - Not Found
```

Is a Zpool, this exis is expected but:

```
blk1:\> load fs0:\drivers\zfs_x64.efi
load: Image fs0:\drivers\zfs_x64.efi loaded at BA821000 - Success

blk1:\> map -r
Device mapping table
  fs0   :HardDisk - Alias hd20b blk0
         pciRoot(0x0)/.*/NVMe(.*)/HD(1,GPT,PARTUUID)
  fs1   :HardDisk - Alias hd20c blk1
         pciRoot(0x0)/.*/NVMe(.*)/HD(2,GPT,PARTUUID)
  blk0  :HardDisk - Alias hd20b fs0
         pciRoot(0x0)/.*/NVMe(.*)/HD(1,GPT,PARTUUID)
  blk1  :HardDisk - Alias fs1 hd20c
         pciRoot(0x0)/.*/NVMe(.*)/HD(2,GPT,PARTUUID)
  hd20b :HardDisk - Alias fs0 blk0
         pciRoot(0x0)/.*/NVMe(.*)/HD(1,GPT,PARTUUID)
  hd20c :HardDisk - Alias fs1 blk1
         pciRoot(0x0)/.*/NVMe(.*)/HD(2,GPT,PARTUUID)

blk1:\> ls
Directory of: blk1:\

  04/10/20  04:14p <DIR> r               0  @
  04/10/20  04:32p <DIR> r               0  root
  04/10/20  04:34p <DIR> r               0  home
  04/10/20  04:35p <DIR> r               0  srv
  04/10/20  04:35p <DIR> r               0  vms
          0 File(s)            0 bytes
          5 Dir(s)


blk1:\>
```

Looks promising...
```
Directory of: blk1:\@

  01/01/70  12:00a <DIR> r               0
  01/01/70  12:00a <DIR> r               0
  01/01/70  12:00a <DIR> r               0
  01/01/70  12:00a <DIR> r               0
  01/01/70  12:00a <DIR> r               0
  01/01/70  12:00a <DIR> r               0
  01/01/70  12:00a <DIR> r               0
          7 File(s)            0 bytes
          0 Dir(s)


blk1:\>
```

Weird, there is no files on this dataset...
```
blk1:\> ls stray-dog\
ls/dir: Cannot read from directory blk1:\stray-dog - Volume Corrupted
ls/dir: Cannot access directory blk1:\stray-dog

blk1:\>
```

Any dataset I created ended with this error. As I think this drivers is based on a GRUB implementation I tryed to do a zpool compatible with GRUB as well using [Archlinux wiki info](https://wiki.archlinux.org/index.php/ZFS#GRUB-compatible_pool_creation)
```
# zpool create -d -o feature@allocation_classes=enabled \
                  -o feature@async_destroy=enabled      \
                  -o feature@bookmarks=enabled          \
                  -o feature@embedded_data=enabled      \
                  -o feature@empty_bpobj=enabled        \
                  -o feature@enabled_txg=enabled        \
                  -o feature@extensible_dataset=enabled \
                  -o feature@filesystem_limits=enabled  \
                  -o feature@hole_birth=enabled         \
                  -o feature@large_blocks=enabled       \
                  -o feature@lz4_compress=enabled       \
                  -o feature@project_quota=enabled      \
                  -o feature@resilver_defer=enabled     \
                  -o feature@spacemap_histogram=enabled \
                  -o feature@spacemap_v2=enabled        \
                  -o feature@userobj_accounting=enabled \
                  -o feature@zpool_checkpoint=enabled   \
                  $POOL_NAME $VDEVS
```

Just, if like me, you wondered too, here are `@` contents:
```
blk1:\> cd @
blk1:\@> type *
type: Target blk1:\@\@ is a directory
type: Target blk1:\@\@ is a directory
type: Target blk1:\@\@ is a directory
type: Target blk1:\@\@ is a directory
type: Target blk1:\@\@ is a directory
type: Target blk1:\@\@ is a directory
type: Target blk1:\@\@ is a directory

blk1:\@> cd @
cd: Target directory not found

blk1:\@>
```

## Drivers

Efi drivers where downloaded from [Akeo](http://efi.akeo.ie/downloads/efifs-1.4/x64/)

## Now what?

GPT partitions, GRUB Bios partition of 1Gb size so I could later still make EFI tests, MBR boot and ZFS using the disk remaining.
