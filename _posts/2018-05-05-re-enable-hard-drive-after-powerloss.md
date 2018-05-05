---
layout: post
title: How to Re-enable Your Hard Drive When Suddenly You Have Powerloss
comments: true
---

If you accidentally have a power loss when you are still using your external hard drive, copying somthing and
the drive can't be detected when you're Mac is back on.

First you can can hit Cmd + Space and type **Disk Utility** and choose it.

1. If you can find your external hard drive on Disk Utility, then pick the partition name, then click the **First Aid** button on the menu. Wait for a minute then your drive is back.

2. If you can't. Then Cmd + Space again and type Terminal. Choose Terminal. Then type **diskutil list**, enter.

you should see something like this:

```
⇒  diskutil list
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *251.0 GB   disk0
   1:                        EFI EFI                     209.7 MB   disk0s1
   2:          Apple_CoreStorage Macintosh HD            250.1 GB   disk0s2
   3:                 Apple_Boot Recovery HD             650.0 MB   disk0s3

/dev/disk1 (internal, virtual):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:                  Apple_HFS Macintosh HD           +249.8 GB   disk1
                                 Logical Volume on disk0s2
                                 E9DFCD4D-790F-4129-8D56-6CA02CB070BE
                                 Unencrypted

/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *1.0 TB     disk2
   1:               Windows_NTFS Mamazo                  1.0 TB     disk2s1
```

if you can see your external hard drive, then your disk is probably still good. Mine is on `/dev/disk2`

Now you run:

`diskutil unmountDisk /dev/disk2`

it will unmount your disk stop any process your hard drive is into. Now you can go to **Disk Utility** again and do as `step 1`

There you go.
