# otus_linux_pro_hw_06

============================================================
Работа с LVM
Цель:
создавать и работать с логическими томами;

Описание/Пошаговая инструкция выполнения домашнего задания:
Для выполнения домашнего задания используйте методичку

Работа с LVM

Что нужно сделать?
на имеющемся образе (centos/7 1804.2)
https://gitlab.com/otus_linux/stands-03-lvm

/dev/mapper/VolGroup00-LogVol00 38G 738M 37G 2% /

    уменьшить том под / до 8G
    выделить том под /home
    выделить том под /var (/var - сделать в mirror)
    для /home - сделать том для снэпшотов
    прописать монтирование в fstab (попробовать с разными опциями и разными файловыми системами на выбор)
    Работа со снапшотами:

    сгенерировать файлы в /home/
    снять снэпшот
    удалить часть файлов
    восстановиться со снэпшота

    (залоггировать работу можно утилитой script, скриншотами и т.п.)

    Задание со звездочкой*
    на нашей куче дисков попробовать поставить btrfs/zfs:
    с кешем и снэпшотами
    разметить здесь каталог /opt
===============================================================

#make vm
vagrant up
vagrant ssh


#look for block devices
[vagrant@lvm ~]$ lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk 
├─sda1                    8:1    0    1M  0 part 
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part 
  ├─VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /
  └─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0   10G  0 disk 
sdc                       8:32   0    2G  0 disk 
sdd                       8:48   0    1G  0 disk 
sde                       8:64   0    1G  0 disk 



#ch user to su
[vagrant@lvm ~]$ sudo -i


#look for disks
[vagrant@lvm ~]$ lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk 
├─sda1                    8:1    0    1M  0 part 
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part 
  ├─VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /
  └─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0   10G  0 disk 
sdc                       8:32   0    2G  0 disk 
sdd                       8:48   0    1G  0 disk 
sde                       8:64   0    1G  0 disk 
[vagrant@lvm ~]$ sudo -i
[root@lvm ~]# 
[root@lvm ~]# 
[root@lvm ~]# lvmdiskscan 
  /dev/VolGroup00/LogVol00 [     <37.47 GiB] 
  /dev/VolGroup00/LogVol01 [       1.50 GiB] 
  /dev/sda2                [       1.00 GiB] 
  /dev/sda3                [     <39.00 GiB] LVM physical volume
  /dev/sdb                 [      10.00 GiB] 
  /dev/sdc                 [       2.00 GiB] 
  /dev/sdd                 [       1.00 GiB] 
  /dev/sde                 [       1.00 GiB] 
  4 disks
  3 partitions
  0 LVM physical volume whole disks
  1 LVM physical volume


#create phys volume
[root@lvm ~]# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.


#create vol group
[root@lvm ~]# vgcreate otus /dev/sdb
  Volume group "otus" successfully created

 
#create logic vol whith 80% free size of vol group
[root@lvm ~]# lvcreate -l+80%FREE -n test otus
  Logical volume "test" created.


#show vol group properties
[root@lvm ~]# vgdisplay otus
  --- Volume group ---
  VG Name               otus
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <10.00 GiB
  PE Size               4.00 MiB
  Total PE              2559
  Alloc PE / Size       2047 / <8.00 GiB
  Free  PE / Size       512 / 2.00 GiB
  VG UUID               wOpPxV-ZpSo-UYsP-gzfl-1kpz-A6LB-fYgDPE



#show phys volume name in vol group
[root@lvm ~]# vgdisplay -v otus | grep 'PV N'
  PV Name               /dev/sdb     


#show log vol info
[root@lvm ~]# lvdisplay /dev/otus/test 
  --- Logical volume ---
  LV Path                /dev/otus/test
  LV Name                test
  VG Name                otus
  LV UUID                SRJrsQ-1b26-IN0r-tAKE-b9d0-m6NM-ALxGF0
  LV Write Access        read/write
  LV Creation host, time lvm, 2024-06-27 20:06:44 +0000
  LV Status              available
  # open                 0
  LV Size                <8.00 GiB
  Current LE             2047
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2



#some info commands about vg pv lv
[root@lvm ~]# vgs
  VG         #PV #LV #SN Attr   VSize   VFree
  VolGroup00   1   2   0 wz--n- <38.97g    0 
  otus         1   1   0 wz--n- <10.00g 2.00g

[root@lvm ~]# lvs
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00 VolGroup00 -wi-ao---- <37.47g                                                    
  LogVol01 VolGroup00 -wi-ao----   1.50g                                                    
  test     otus       -wi-a-----  <8.00g                                                     
[root@lvm ~]# pvs
  PV         VG         Fmt  Attr PSize   PFree
  /dev/sda3  VolGroup00 lvm2 a--  <38.97g    0 
  /dev/sdb   otus       lvm2 a--  <10.00g 2.00g




#create log vol 100mb fixed size in same vol group
[root@lvm ~]~# lvcreate -L100M -n small otus 
  Logical volume "small" created.


[root@lvm ~]# lvs
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00 VolGroup00 -wi-ao---- <37.47g                                                    
  LogVol01 VolGroup00 -wi-ao----   1.50g                                                    
  small    otus       -wi-a----- 100.00m                                                    
  test     otus       -wi-a-----  <8.00g 


#create and mount fs
[root@lvm ~]# mkfs.ext4 /dev/otus/test
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
524288 inodes, 2096128 blocks
104806 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=2147483648
64 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
  32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done 


[root@lvm ~]# mkdir /data && mount /dev/otus/test /data

[root@lvm ~]# mount | grep /data
/dev/mapper/otus-test on /data type ext4 (rw,relatime,seclabel,data=ordered)



#extend vol group
[root@lvm ~]# pvs
  PV         VG         Fmt  Attr PSize   PFree
  /dev/sda3  VolGroup00 lvm2 a--  <38.97g    0 
  /dev/sdb   otus       lvm2 a--  <10.00g 1.90g


[root@lvm ~]# pvcreate /dev/sdc
  Physical volume "/dev/sdc" successfully created.

[root@lvm ~]# vgextend otus /dev/sdc
  Volume group "otus" successfully extende

[root@lvm ~]# vgs
  VG         #PV #LV #SN Attr   VSize   VFree 
  VolGroup00   1   2   0 wz--n- <38.97g     0 
  otus         2   2   0 wz--n-  11.99g <3.90g

[root@lvm ~]# vgdisplay -v otus | grep 'PV N'
  PV Name               /dev/sdb     
  PV Name               /dev/sdc 


#filling disk space
[root@lvm ~]# dd if=/dev/zero of=/data/test.log bs=1M \
>  count=8000 status=progress
7734296576 bytes (7.7 GB) copied, 6.014680 s, 1.3 GB/s
dd: error writing ‘/data/test.log’: No space left on device
7880+0 records in
7879+0 records out
8262189056 bytes (8.3 GB) copied, 6.46906 s, 1.3 GB/s



#show free space
[root@lvm ~]# df -Th /data/
Filesystem            Type  Size  Used Avail Use% Mounted on
/dev/mapper/otus-test ext4  7.8G  7.8G     0 100% /data


#extend log volume
[root@lvm ~]# lvextend -l+80%FREE /dev/otus/test
  Size of logical volume otus/test changed from <8.00 GiB (2047 extents) to <11.12 GiB (2846 extents).
  Logical volume otus/test successfully resized.


[root@lvm ~]# lvs
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00 VolGroup00 -wi-ao---- <37.47g                                                    
  LogVol01 VolGroup00 -wi-ao----   1.50g                                                    
  small    otus       -wi-a----- 100.00m                                                    
  test     otus       -wi-ao---- <11.12g                                                     
[root@lvm ~]# lvs /dev/otus/test 
  LV   VG   Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  test otus -wi-ao---- <11.12g 


#resize fs (extend)
[root@lvm ~]# df -Th /data
Filesystem            Type  Size  Used Avail Use% Mounted on
/dev/mapper/otus-test ext4  7.8G  7.8G     0 100% /data

[root@lvm ~]# resize2fs /dev/otus/test
resize2fs 1.42.9 (28-Dec-2013)
Filesystem at /dev/otus/test is mounted on /data; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 2
The filesystem on /dev/otus/test is now 2914304 blocks long.


[root@lvm ~]# df -Th /data
Filesystem            Type  Size  Used Avail Use% Mounted on
/dev/mapper/otus-test ext4   11G  7.8G  2.6G  76% /data


#reduce lv
[root@lvm ~]# umount /data/
[root@lvm ~]# e2fsck -fy /dev/otus/test
e2fsck 1.42.9 (28-Dec-2013)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/otus/test: 12/729088 files (0.0% non-contiguous), 2105907/2914304 blocks
[root@lvm ~]# resize2fs /dev/otus/test 10G
resize2fs 1.42.9 (28-Dec-2013)
Resizing the filesystem on /dev/otus/test to 2621440 (4k) blocks.
The filesystem on /dev/otus/test is now 2621440 blocks long.


[root@lvm ~]# lvreduce /dev/otus/test -L 10G
  WARNING: Reducing active logical volume to 10.00 GiB.
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce otus/test? [y/n]: y
  Size of logical volume otus/test changed from <11.12 GiB (2846 extents) to 10.00 GiB (2560 extents).
  Logical volume otus/test successfully resized.
[root@lvm ~]# mount /dev/otus/test /data/



[root@lvm ~]# df -Th /data
Filesystem            Type  Size  Used Avail Use% Mounted on
/dev/mapper/otus-test ext4  9.8G  7.8G  1.6G  84% /data
[root@lvm ~]# lvs /dev/otus/test 
  LV   VG   Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  test otus -wi-ao---- 10.00g    

  #snapshots
#create snapshot

[root@lvm ~]# lvcreate -L 500M -s -n test-snap /dev/otus/test
  Logical volume "test-snap" created.
[root@lvm ~]# vgs
  VG         #PV #LV #SN Attr   VSize   VFree 
  VolGroup00   1   2   0 wz--n- <38.97g     0 
  otus         2   3   1 wz--n-  11.99g <1.41g
[root@lvm ~]# vgs -o +lv_size,lv_name | grep test
  otus         2   3   1 wz--n-  11.99g <1.41g  10.00g test     
  otus         2   3   1 wz--n-  11.99g <1.41g 500.00m test-snap



#check disks
[root@lvm ~]# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk 
├─sda1                    8:1    0    1M  0 part 
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part 
  ├─VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /
  └─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0   10G  0 disk 
├─otus-small            253:3    0  100M  0 lvm  
└─otus-test-real        253:4    0   10G  0 lvm  
  ├─otus-test           253:2    0   10G  0 lvm  /data
  └─otus-test--snap     253:6    0   10G  0 lvm  
sdc                       8:32   0    2G  0 disk 
├─otus-test-real        253:4    0   10G  0 lvm  
│ ├─otus-test           253:2    0   10G  0 lvm  /data
│ └─otus-test--snap     253:6    0   10G  0 lvm  
└─otus-test--snap-cow   253:5    0  500M  0 lvm  
  └─otus-test--snap     253:6    0   10G  0 lvm  
sdd                       8:48   0    1G  0 disk 
sde                       8:64   0    1G  0 disk 



#mount snap as disk
[root@lvm ~]# mkdir  /data_snap
[root@lvm ~]# mount /dev/otus/test-snap /data-snap/
mount: mount point /data-snap/ does not exist
[root@lvm ~]# rmdir  /data_snap
[root@lvm ~]# mkdir  /data_snap && mount /dev/otus/test-snap /data_snap/
[root@lvm ~]# ll /data_snap/
total 8068564
drwx------. 2 root root      16384 Jun 27 20:14 lost+found
-rw-r--r--. 1 root root 8262189056 Jun 27 20:18 test.log
[root@lvm ~]# umount /data_snap 




#test snap removeing log
[root@lvm ~]# rm /data/test.log
rm: remove regular file ‘/data/test.log’? y
[root@lvm ~]# ls /data
lost+found
[root@lvm ~]# umount /data
[root@lvm ~]# lvconvert --merge /dev/otus/test-snap
  Merging of volume otus/test-snap started.
  otus/test: Merged: 99.99%


[root@lvm ~]# mount /dev/otus/test /data
[root@lvm ~]# ls -al /data
total 8068568
drwxr-xr-x.  3 root root       4096 Jun 27 20:18 .
dr-xr-xr-x. 20 root root        268 Jun 27 20:28 ..
drwx------.  2 root root      16384 Jun 27 20:14 lost+found
-rw-r--r--.  1 root root 8262189056 Jun 27 20:18 test.log




#lvm raid
[root@lvm ~]# pvcreate /dev/sd{d,e}
  Physical volume "/dev/sdd" successfully created.
  Physical volume "/dev/sde" successfully created.
[root@lvm ~]# vgcreate vg0 /dev/sd{d,e}
  Volume group "vg0" successfully created
[root@lvm ~]# lvcreate -l+80%FREE -m1 -n mirror vg0
  Logical volume "mirror" created.
[root@lvm ~]# lvs
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00 VolGroup00 -wi-ao---- <37.47g                                                    
  LogVol01 VolGroup00 -wi-ao----   1.50g                                                    
  small    otus       -wi-a----- 100.00m                                                    
  test     otus       -wi-ao----  10.00g                                                    
  mirror   vg0        rwi-a-r--- 816.00m                                    100.00 





