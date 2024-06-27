
Домашнее задание
На имеющемся образе centos/7 - v. 1804.2
Уменьшить том под / до 8G.
Выделить том под /home.
Выделить том под /var - сделать в mirror.
/home - сделать том для снапшотов.
Прописать монтирование в fstab. Попробовать с разными опциями и разными файловыми системами (на выбор).
Работа со снапшотами:
сгенерить файлы в /home/;
снять снапшот;
удалить часть файлов;
восстановится со снапшота.
* На дисках попробовать поставить btrfs/zfs — с кешем, снапшотами и разметить там каталог /opt.
========================


#reduce / to 8gb
vagrant up
vagrant ssh
sudo yum install xfsdump
sudo yum install lvm2

sudo -i

#create new lvfor new root
[root@lvm ~]# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
[root@lvm ~]# vgcreate vg_root /dev/sdb
  Volume group "vg_root" successfully created
[root@lvm ~]# lvcreate -n lv_root -l +100%FREE /dev/vg_root
  Logical volume "lv_root" created.
[root@lvm ~]# lsblk
NAME              MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sda                 8:0    0  40G  0 disk 
└─sda1              8:1    0  40G  0 part /
sdb                 8:16   0  10G  0 disk 
└─vg_root-lv_root 253:0    0  10G  0 lvm  
sdc                 8:32   0   2G  0 disk 
sdd                 8:48   0   1G  0 disk 
sde                 8:64   0   1G  0 disk 


[root@lvm ~]# mkfs.xfs /dev/vg_root/lv_root 
meta-data=/dev/vg_root/lv_root   isize=512    agcount=4, agsize=655104 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=2620416, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0


[root@lvm ~]# mount /dev/vg_root/lv_root /mnt/


#copy old root to new lv
[root@lvm ~]# xfsdump -J - /dev/sda1 | xfsrestore -j - /mnt
...
...
...
xfsdump: dump size (non-dir files) : 3336405856 bytes
xfsdump: dump complete: 11 seconds elapsed
xfsdump: Dump Status: SUCCESS
xfsrestore: restore complete: 11 seconds elapsed
xfsrestore: Restore Status: SUCCESS

#check it
[root@lvm ~]# ls /mnt/
bin   dev  home  lib64  mnt  proc  run   srv       sys  usr      var
boot  etc  lib   media  opt  root  sbin  swapfile  tmp  vagrant
[root@lvm ~]# ls /
bin   dev  home  lib64  mnt  proc  run   srv       sys  usr      var
boot  etc  lib   media  opt  root  sbin  swapfile  tmp  vagrant


#reconfigure grub to new /
[root@lvm ~]# for i in /proc/ /sys/ /dev/ /run/ /boot/; \
> do mount --bind $i /mnt/$i; done
[root@lvm ~]# chroot /mnt/
[root@lvm /]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-1127.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-1127.el7.x86_64.img
done

#renew initrd
[root@lvm /]# cd /boot ; for i in `ls initramfs-*img`; \
> do dracut -v $i `echo $i|sed "s/initramfs-//g; \
> > s/.img//g"` --force; done
...
...
...
*** Store current command line parameters ***
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-1127.el7.x86_64.img' done ***


#edit grub.cfg
[root@lvm boot]# lsblk
NAME              MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sda                 8:0    0  40G  0 disk 
└─sda1              8:1    0  40G  0 part /boot
sdb                 8:16   0  10G  0 disk 
└─vg_root-lv_root 253:0    0  10G  0 lvm  /
sdc                 8:32   0   2G  0 disk 
sdd                 8:48   0   1G  0 disk 
sde                 8:64   0   1G  0 disk 










