# Домашнее задание к занятию "3.5. Файловые системы"
## Лекция 1

1. Узнайте о [sparse](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB) (разряженных) файлах.

    ```bash 
    сжатый файл, в котором нулевые последовательности в файле заменяются на метаданные (информации об эти последовательностях) 
   ```

1. Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?

    ```bash
    Нет, права хранятся в inode, а хардлинк это ссылка на него. 
   ```

1. Сделайте `vagrant destroy` на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:

    ```bash
    Vagrant.configure("2") do |config|
      config.vm.box = "bento/ubuntu-20.04"
      config.vm.provider :virtualbox do |vb|
        lvm_experiments_disk0_path = "/tmp/lvm_experiments_disk0.vmdk"
        lvm_experiments_disk1_path = "/tmp/lvm_experiments_disk1.vmdk"
        vb.customize ['createmedium', '--filename', lvm_experiments_disk0_path, '--size', 2560]
        vb.customize ['createmedium', '--filename', lvm_experiments_disk1_path, '--size', 2560]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk0_path]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk1_path]
      end
    end
    ```

   Данная конфигурация создаст новую виртуальную машину с двумя дополнительными неразмеченными дисками по 2.5 Гб.

    ```bash
   vagrant@vagrant:~$ lsblk
    NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    loop0                       7:0    0 61.9M  1 loop /snap/core20/1328
    loop1                       7:1    0 67.2M  1 loop /snap/lxd/21835
    loop2                       7:2    0 43.6M  1 loop /snap/snapd/14978
    sda                         8:0    0   64G  0 disk
    ├─sda1                      8:1    0    1M  0 part
    ├─sda2                      8:2    0  1.5G  0 part /boot
    └─sda3                      8:3    0 62.5G  0 part
    └─ubuntu--vg-ubuntu--lv 253:0    0 31.3G  0 lvm  /
    sdb                         8:16   0  2.5G  0 disk
    sdc                         8:32   0  2.5G  0 disk 
   ```

1. Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.

    ```bash
     vagrant@vagrant:~$ sudo fdisk /dev/sdb

    Welcome to fdisk (util-linux 2.34).
    Changes will remain in memory only, until you decide to write them.
    Be careful before using the write command.
    
    Device does not contain a recognized partition table.
    Created a new DOS disklabel with disk identifier 0x6dec3869.
    
    Command (m for help): n
    Partition type
    p   primary (0 primary, 0 extended, 4 free)
    e   extended (container for logical partitions)
    Select (default p):
    
    Using default response p.
    Partition number (1-4, default 1):
    First sector (2048-5242879, default 2048):
    Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-5242879, default 5242879): +2G
    
    Created a new partition 1 of type 'Linux' and of size 2 GiB.
    
    Command (m for help): n
    Partition type
    p   primary (1 primary, 0 extended, 3 free)
    e   extended (container for logical partitions)
    Select (default p):
    
    Using default response p.
    Partition number (2-4, default 2):
    First sector (4196352-5242879, default 4196352):
    Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-5242879, default 5242879):
    
    Created a new partition 2 of type 'Linux' and of size 511 MiB.
    
    Command (m for help): w
    The partition table has been altered.
    Calling ioctl() to re-read partition table.
    Syncing disks.
   
    vagrant@vagrant:~$ lsblk
    NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    loop0                       7:0    0 61.9M  1 loop /snap/core20/1328
    loop1                       7:1    0 67.2M  1 loop /snap/lxd/21835
    loop2                       7:2    0 43.6M  1 loop /snap/snapd/14978
    loop3                       7:3    0   47M  1 loop /snap/snapd/16292
    loop4                       7:4    0   62M  1 loop /snap/core20/1611
    loop5                       7:5    0 67.8M  1 loop /snap/lxd/22753
    sda                         8:0    0   64G  0 disk
    ├─sda1                      8:1    0    1M  0 part
    ├─sda2                      8:2    0  1.5G  0 part /boot
    └─sda3                      8:3    0 62.5G  0 part
    └─ubuntu--vg-ubuntu--lv 253:0    0 31.3G  0 lvm  /
    sdb                         8:16   0  2.5G  0 disk
    ├─sdb1                      8:17   0    2G  0 part
    └─sdb2                      8:18   0  511M  0 part
    sdc                         8:32   0  2.5G  0 disk
   ```

1. Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.

    ```bash
    root@vagrant:~# sfdisk -d /dev/sdb | sfdisk /dev/sdc 
    
    root@vagrant:~# lsblk
    NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    loop0                       7:0    0 61.9M  1 loop /snap/core20/1328
    loop1                       7:1    0 67.2M  1 loop /snap/lxd/21835
    loop2                       7:2    0 43.6M  1 loop /snap/snapd/14978
    loop3                       7:3    0   47M  1 loop /snap/snapd/16292
    loop4                       7:4    0   62M  1 loop /snap/core20/1611
    loop5                       7:5    0 67.8M  1 loop /snap/lxd/22753
    sda                         8:0    0   64G  0 disk
    ├─sda1                      8:1    0    1M  0 part
    ├─sda2                      8:2    0  1.5G  0 part /boot
    └─sda3                      8:3    0 62.5G  0 part
    └─ubuntu--vg-ubuntu--lv 253:0    0 31.3G  0 lvm  /
    sdb                         8:16   0  2.5G  0 disk
    ├─sdb1                      8:17   0    2G  0 part
    └─sdb2                      8:18   0  511M  0 part
    sdc                         8:32   0  2.5G  0 disk
    ├─sdc1                      8:33   0    2G  0 part
    └─sdc2                      8:34   0  511M  0 part
   ```

1. Соберите `mdadm` RAID1 на паре разделов 2 Гб.

    ```bash
    root@vagrant:~# mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sd[bc]1 
   ```

1. Соберите `mdadm` RAID0 на второй паре маленьких разделов.

    ```bash
    mdadm --create /dev/md1 --level=0 --raid-devices=2 /dev/sd[bc]2 
   ```

1. Создайте 2 независимых PV на получившихся md-устройствах.

    ```bash
    root@vagrant:~# pvcreate /dev/md[01]
    Physical volume "/dev/md0" successfully created.
    Physical volume "/dev/md1" successfully created.
    root@vagrant:~# pvs
    PV         VG        Fmt  Attr PSize    PFree
    /dev/md0             lvm2 ---    <2.00g   <2.00g
    /dev/md1             lvm2 ---  1018.00m 1018.00m
    /dev/sda3  ubuntu-vg lvm2 a--   <62.50g   31.25g 
   ```

1. Создайте общую volume-group на этих двух PV.

    ```bash
    root@vagrant:~# vgcreate vg01 /dev/md0 /dev/md1
    Volume group "vg01" successfully created
    root@vagrant:~# vgs
    VG        #PV #LV #SN Attr   VSize   VFree
    ubuntu-vg   1   1   0 wz--n- <62.50g 31.25g
    vg01        2   0   0 wz--n-  <2.99g <2.99g 
   ```

1. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.

    ```bash
    root@vagrant:~# lvcreate -L 100m -n vg01-lv vg01 /dev/md1
    Logical volume "vg01-lv" created.
    root@vagrant:~# lvs -o +devices
    LV        VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert Devices
    ubuntu-lv ubuntu-vg -wi-ao---- <31.25g                                                     /dev/sda3(0)
    vg01-lv   vg01      -wi-a----- 100.00m                                                     /dev/md1(0) 
   ```

1. Создайте `mkfs.ext4` ФС на получившемся LV.

    ```bash
    root@vagrant:~# mkfs.ext4 /dev/vg01/vg01-lv
    mke2fs 1.45.5 (07-Jan-2020)
    Creating filesystem with 25600 4k blocks and 25600 inodes
    
    Allocating group tables: done
    Writing inode tables: done
    Creating journal (1024 blocks): done
    Writing superblocks and filesystem accounting information: done 
   ```

1. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.

    ```bash
    root@vagrant:~# mkdir /tmp/new
    root@vagrant:~# mount /dev/vg01/vg01-lv /tmp/new 
   ```

1. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.

    ```bash
    root@vagrant:~# wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz
    --2022-08-22 12:46:27--  https://mirror.yandex.ru/ubuntu/ls-lR.gz
    Resolving mirror.yandex.ru (mirror.yandex.ru)... 213.180.204.183, 2a02:6b8::183
    Connecting to mirror.yandex.ru (mirror.yandex.ru)|213.180.204.183|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 22290833 (21M) [application/octet-stream]
    Saving to: ‘/tmp/new/test.gz’
    
    /tmp/new/test.gz         100%[==================================>]  21.26M  4.34MB/s    in 5.0s
    
    2022-08-22 12:46:32 (4.23 MB/s) - ‘/tmp/new/test.gz’ saved [22290833/22290833] 
   ```

1. Прикрепите вывод `lsblk`.

    ```bash
    root@vagrant:~# lsblk
    NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
    loop0                       7:0    0 61.9M  1 loop  /snap/core20/1328
    loop1                       7:1    0 67.2M  1 loop  /snap/lxd/21835
    loop2                       7:2    0 43.6M  1 loop  /snap/snapd/14978
    loop3                       7:3    0   47M  1 loop  /snap/snapd/16292
    loop4                       7:4    0   62M  1 loop  /snap/core20/1611
    loop5                       7:5    0 67.8M  1 loop  /snap/lxd/22753
    sda                         8:0    0   64G  0 disk
    ├─sda1                      8:1    0    1M  0 part
    ├─sda2                      8:2    0  1.5G  0 part  /boot
    └─sda3                      8:3    0 62.5G  0 part
      └─ubuntu--vg-ubuntu--lv 253:0    0 31.3G  0 lvm   /
    sdb                         8:16   0  2.5G  0 disk
    ├─sdb1                      8:17   0    2G  0 part
    │ └─md0                     9:0    0    2G  0 raid1
    └─sdb2                      8:18   0  511M  0 part
      └─md1                     9:1    0 1018M  0 raid0
        └─vg01-vg01--lv       253:1    0  100M  0 lvm   /tmp/new
    sdc                         8:32   0  2.5G  0 disk
    ├─sdc1                      8:33   0    2G  0 part
    │ └─md0                     9:0    0    2G  0 raid1
    └─sdc2                      8:34   0  511M  0 part
      └─md1                     9:1    0 1018M  0 raid0
        └─vg01-vg01--lv       253:1    0  100M  0 lvm   /tmp/new 
   ```

1. Протестируйте целостность файла:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0 
   ```

1. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.

    ```bash
    root@vagrant:~# pvmove -n vg01-lv /dev/md1 /dev/md0
    /dev/md1: Moved: 12.00%
    /dev/md1: Moved: 100.00%
    root@vagrant:~# lsblk
    NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
    loop0                       7:0    0 61.9M  1 loop  /snap/core20/1328
    loop1                       7:1    0 67.2M  1 loop  /snap/lxd/21835
    loop2                       7:2    0 43.6M  1 loop  /snap/snapd/14978
    loop3                       7:3    0   47M  1 loop  /snap/snapd/16292
    loop4                       7:4    0   62M  1 loop  /snap/core20/1611
    loop5                       7:5    0 67.8M  1 loop  /snap/lxd/22753
    sda                         8:0    0   64G  0 disk
    ├─sda1                      8:1    0    1M  0 part
    ├─sda2                      8:2    0  1.5G  0 part  /boot
    └─sda3                      8:3    0 62.5G  0 part
      └─ubuntu--vg-ubuntu--lv 253:0    0 31.3G  0 lvm   /
    sdb                         8:16   0  2.5G  0 disk
    ├─sdb1                      8:17   0    2G  0 part
    │ └─md0                     9:0    0    2G  0 raid1
    │   └─vg01-vg01--lv       253:1    0  100M  0 lvm   /tmp/new
    └─sdb2                      8:18   0  511M  0 part
      └─md1                     9:1    0 1018M  0 raid0
    sdc                         8:32   0  2.5G  0 disk
    ├─sdc1                      8:33   0    2G  0 part
    │ └─md0                     9:0    0    2G  0 raid1
    │   └─vg01-vg01--lv       253:1    0  100M  0 lvm   /tmp/new
    └─sdc2                      8:34   0  511M  0 part
      └─md1                     9:1    0 1018M  0 raid0 
   ```

1. Сделайте `--fail` на устройство в вашем RAID1 md.

    ```bash
    root@vagrant:~# mdadm --fail /dev/md0 /dev/sdb1
    mdadm: set /dev/sdb1 faulty in /dev/md0 
   ```

1. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.

    ```bash
    root@vagrant:~# dmesg | grep md0
    [ 5813.689643] md/raid1:md0: not clean -- starting background reconstruction
    [ 5813.689645] md/raid1:md0: active with 2 out of 2 mirrors
    [ 5813.689661] md0: detected capacity change from 0 to 2144337920
    [ 5813.692499] md: resync of RAID array md0
    [ 5824.096378] md: md0: resync done.
    [ 7500.845230] md/raid1:md0: Disk failure on sdb1, disabling device.
    md/raid1:md0: Operation continuing on 1 devices. 
   ```

1. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz && echo $?
    0 
   ```

1. Погасите тестовый хост, `vagrant destroy`.

    ```bash
    vagrant destroy
    default: Are you sure you want to destroy the 'default' VM? [y/N] y
    ==> default: Forcing shutdown of VM...
    ==> default: Destroying VM and associated drives...

   ```