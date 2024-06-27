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


#look for block devices
vagrant@lvm-1:~$ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0  63.3M  1 loop /snap/core20/1879
loop1    7:1    0 111.9M  1 loop /snap/lxd/24322
loop2    7:2    0  53.2M  1 loop /snap/snapd/19122
sda      8:0    0    40G  0 disk 
└─sda1   8:1    0    40G  0 part /
sdb      8:16   0    10M  0 disk 
sdc      8:32   0    10G  0 disk 
sdd      8:48   0     2G  0 disk 
sde      8:64   0     1G  0 disk 
sdf      8:80   0     1G  0 disk 


#ch user to su
vagrant@lvm-1:~$ sudo -i


#look for disks
root@lvm-1:~# lvmdiskscan
  /dev/loop0 [     <63.34 MiB] 
  /dev/loop1 [    <111.95 MiB] 
  /dev/sda1  [     <40.00 GiB] 
  /dev/loop2 [     <53.24 MiB] 
  /dev/sdb   [      10.00 MiB] 
  /dev/sdc   [      10.00 GiB] 
  /dev/sdd   [       2.00 GiB] 
  /dev/sde   [       1.00 GiB] 
  /dev/sdf   [       1.00 GiB] 
  5 disks
  4 partitions
  0 LVM physical volume whole disks
  0 LVM physical volumes

#create phys volume
root@lvm-1:~# pvcreate /dev/sdc
  Physical volume "/dev/sdc" successfully created.

#create vol group
root@lvm-1:~# vgcreate otus /dev/sdc 
  Volume group "otus" successfully created
 
#create logic vol whith 80% free size of vol group
root@lvm-1:~# lvcreate -l +80%FREE -n test otus
  Logical volume "test" created.

#show vol group properties
root@lvm-1:~# vgdisplay otus 
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
  VG UUID               er26kg-KAVM-6bxS-gXzV-d0l7-NeD3-yhrnzP


#show phys volume name in vol group
root@lvm-1:~# vgdisplay -v otus | grep 'PV N'
  PV Name               /dev/sdc     


#show log vol info
root@lvm-1:~# lvdisplay /dev/otus/test 
  --- Logical volume ---
  LV Path                /dev/otus/test
  LV Name                test
  VG Name                otus
  LV UUID                ir8dy3-Q1o9-PlLJ-gmiO-1Sdh-A4Vh-UJNJOh
  LV Write Access        read/write
  LV Creation host, time lvm-1, 2024-06-07 14:36:39 +0000
  LV Status              available
  # open                 0
  LV Size                <8.00 GiB
  Current LE             2047
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0


#some info commands about vg pv lv
root@lvm-1:~# vgs
  VG   #PV #LV #SN Attr   VSize   VFree
  otus   1   1   0 wz--n- <10.00g 2.00g
root@lvm-1:~# lvs
  LV   VG   Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  test otus -wi-a----- <8.00g                                                    
root@lvm-1:~# pvs
  PV         VG   Fmt  Attr PSize   PFree
  /dev/sdc   otus lvm2 a--  <10.00g 2.00g



#create log vol 100mb fixed size in same vol group
root@lvm-1:~# lvcreate -L100M -n small otus 
  Logical volume "small" created.


  root@lvm-1:~# lvs
  LV    VG   Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  small otus -wi-a----- 100.00m                                                    
  test  otus -wi-a-----  <8.00g  


#create and mount fs
root@lvm-1:~# mkfs.ext4 /dev/otus/
small  test   
root@lvm-1:~# mkfs.ext4 /dev/otus/test 
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 2096128 4k blocks and 524288 inodes
Filesystem UUID: ce3ec60f-e50e-4f1a-9df7-d0c65f1e9cc7
Superblock backups stored on blocks: 
    32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 

root@lvm-1:~# mkdir /data && mount /dev/otus/test /data

root@lvm-1:~# mount | grep /data
/dev/mapper/otus-test on /data type ext4 (rw,relatime)


#extend vol group

root@lvm-1:~# pvs
  PV         VG   Fmt  Attr PSize   PFree
  /dev/sdc   otus lvm2 a--  <10.00g 1.90g

root@lvm-1:~# pvcreate /dev/sdd
  Physical volume "/dev/sdd" successfully created.

root@lvm-1:~# vgextend otus /dev/sdd
  Volume group "otus" successfully extended

root@lvm-1:~# vgs
  VG   #PV #LV #SN Attr   VSize  VFree 
  otus   2   2   0 wz--n- 11.99g <3.90g
root@lvm-1:~# vgdisplay -v otus | grep 'PV N'
  PV Name               /dev/sdc     
  PV Name               /dev/sdd 

#filling disk space
root@lvm-1:~#  dd if=/dev/zero of=/data/test.log bs=1M \
 count=8000 status=progress
1224736768 bytes (1.2 GB, 1.1 GiB) copied, 1 s, 1.2 GB/s
7789871104 bytes (7.8 GB, 7.3 GiB) copied, 6 s, 1.3 GB/s
dd: error writing '/data/test.log': No space left on device
7944+0 records in
7943+0 records out
8329297920 bytes (8.3 GB, 7.8 GiB) copied, 6.46246 s, 1.3 GB/s


#show free space
root@lvm-1:~#  df -Th /data/
Filesystem            Type  Size  Used Avail Use% Mounted on
/dev/mapper/otus-test ext4  7.8G  7.8G     0 100% /data


#extend log volume
root@lvm-1:~#  lvextend -l+80%FREE /dev/otus/test
  Size of logical volume otus/test changed from <8.00 GiB (2047 extents) to <11.12 GiB (2846 extents).
  Logical volume otus/test successfully resized.

root@lvm-1:~# lvs
  LV    VG   Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  small otus -wi-a----- 100.00m                                                    
  test  otus -wi-ao---- <11.12g                                                    
root@lvm-1:~# lvs /dev/otus/test 
  LV   VG   Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  test otus -wi-ao---- <11.12g     


#resize fs (extend)
root@lvm-1:~# df -Th /data
Filesystem            Type  Size  Used Avail Use% Mounted on
/dev/mapper/otus-test ext4  7.8G  7.8G     0 100% /data
root@lvm-1:~# resize2fs /dev/otus/test
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/otus/test is mounted on /data; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 2
The filesystem on /dev/otus/test is now 2914304 (4k) blocks long.

root@lvm-1:~# df -Th /data
Filesystem            Type  Size  Used Avail Use% Mounted on
/dev/mapper/otus-test ext4   11G  7.8G  2.6G  76% /data

#reduce lv
root@lvm-1:~# umount /data/
root@lvm-1:~# e2fsck -fy /dev/otus/test 
e2fsck 1.46.5 (30-Dec-2021)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/otus/test: 12/729088 files (0.0% non-contiguous), 2105907/2914304 blocks
root@lvm-1:~# resize2fs /dev/otus/test 10G
resize2fs 1.46.5 (30-Dec-2021)
Resizing the filesystem on /dev/otus/test to 2621440 (4k) blocks.
The filesystem on /dev/otus/test is now 2621440 (4k) blocks long.

root@lvm-1:~# lvreduce /dev/otus/test -L 10G
  WARNING: Reducing active logical volume to 10.00 GiB.
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce otus/test? [y/n]: y
  Size of logical volume otus/test changed from <11.12 GiB (2846 extents) to 10.00 GiB (2560 extents).
  Logical volume otus/test successfully resized.
root@lvm-1:~# mount /dev/otus/test /data/


root@lvm-1:~# df -Th /data
Filesystem            Type  Size  Used Avail Use% Mounted on
/dev/mapper/otus-test ext4  9.8G  7.8G  1.6G  84% /data
root@lvm-1:~# lvs /dev/otus/test 
  LV   VG   Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  test otus -wi-ao---- 10.00g 

  #snapshots
#create snapshot

root@lvm-1:~# lvcreate -L 500M -s -n test-snap /dev/otus/test
  Logical volume "test-snap" created.
root@lvm-1:~# vgs
  VG   #PV #LV #SN Attr   VSize  VFree 
  otus   2   3   1 wz--n- 11.99g <1.41g
root@lvm-1:~# vgs -o +lv_size,lv_name | grep test
  otus   2   3   1 wz--n- 11.99g <1.41g  10.00g test     
  otus   2   3   1 wz--n- 11.99g <1.41g 500.00m test-snap



#check disks
root@lvm-1:~# lsblk
NAME                  MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0                   7:0    0  63.3M  1 loop /snap/core20/1879
loop1                   7:1    0 111.9M  1 loop /snap/lxd/24322
loop2                   7:2    0  53.2M  1 loop /snap/snapd/19122
sda                     8:0    0    40G  0 disk 
└─sda1                  8:1    0    40G  0 part /
sdb                     8:16   0    10M  0 disk 
sdc                     8:32   0    10G  0 disk 
├─otus-small          253:1    0   100M  0 lvm  
└─otus-test-real      253:2    0    10G  0 lvm  
  ├─otus-test         253:0    0    10G  0 lvm  /data
  └─otus-test--snap   253:4    0    10G  0 lvm  
sdd                     8:48   0     2G  0 disk 
├─otus-test-real      253:2    0    10G  0 lvm  
│ ├─otus-test         253:0    0    10G  0 lvm  /data
│ └─otus-test--snap   253:4    0    10G  0 lvm  
└─otus-test--snap-cow 253:3    0   500M  0 lvm  
  └─otus-test--snap   253:4    0    10G  0 lvm  
sde                     8:64   0     1G  0 disk 
sdf                     8:80   0     1G  0 disk 


#mount snap as disk
root@lvm-1:~# mkdir  /data_snap
root@lvm-1:~#  mount /dev/otus/test-snap /data-snap/
mount: /data-snap/: mount point does not exist.
root@lvm-1:~# rmdir  /data_snap
root@lvm-1:~# mkdir  /data_snap && mount /dev/otus/test-snap /data_snap/
root@lvm-1:~# ll /data_snap/
total 8134108
drwxr-xr-x  3 root root       4096 Jun  7 14:56 ./
drwxr-xr-x 22 root root       4096 Jun  7 15:17 ../
drwx------  2 root root      16384 Jun  7 14:46 lost+found/
-rw-r--r--  1 root root 8329297920 Jun  7 14:56 test.log
root@lvm-1:~# umount /data_snap 



#test snap removeing log
root@lvm-1:~# rm /data/test.log
root@lvm-1:~# 
root@lvm-1:~# 
root@lvm-1:~# ll /data/
total 24
drwxr-xr-x  3 root root  4096 Jun  7 15:20 ./
drwxr-xr-x 22 root root  4096 Jun  7 15:17 ../
drwx------  2 root root 16384 Jun  7 14:46 lost+found/
root@lvm-1:~# 
root@lvm-1:~# umount /data
root@lvm-1:~# 
root@lvm-1:~# lvconvert --merge /dev/otus/test
test       test-snap  
root@lvm-1:~# lvconvert --merge /dev/otus/test-snap 
  Merging of volume otus/test-snap started.
  otus/test: Merged: 100.00%

root@lvm-1:~# 
root@lvm-1:~# mount /dev/otus/test /data
root@lvm-1:~# ll /data
total 8134108
drwxr-xr-x  3 root root       4096 Jun  7 14:56 ./
drwxr-xr-x 22 root root       4096 Jun  7 15:17 ../
drwx------  2 root root      16384 Jun  7 14:46 lost+found/
-rw-r--r--  1 root root 8329297920 Jun  7 14:56 test.log
root@lvm-1:~# 



#lvm raid
root@lvm-1:~# pvcreate /dev/sd{e,f}
  Physical volume "/dev/sde" successfully created.
  Physical volume "/dev/sdf" successfully created.
root@lvm-1:~# vgcreate vg0 /dev/sd{e,f}
  Volume group "vg0" successfully created
root@lvm-1:~# lvcreate -l +80%FREE -m1 -n mirror vg0
  Logical volume "mirror" created.
root@lvm-1:~# lvs
  LV     VG   Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  small  otus -wi-a----- 100.00m                                                    
  test   otus -wi-ao----  10.00g                                                    
  mirror vg0  rwi-a-r--- 816.00m                                    93.86         






