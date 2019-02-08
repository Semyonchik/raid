# raid
Задание на **

В вагрант файле добавил второй диск

<details> <summary>Вывод lsblk:</summary>
	
		NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
		sda      8:0    0  40G  0 disk
		`-sda1   8:1    0  40G  0 part /
		sdb      8:16   0  40G  0 disk
</details>

Скопировал раздел sda на sdb

	sfdisk -d /dev/sda | sfdisk /dev/sdb
	
<details> <summary>Поменял тип раздела sdb на Linux raid autodetect</summary>
	
	#fdisk /dev/sdb
	Welcome to fdisk (util-linux 2.23.2).

	Changes will remain in memory only, until you decide to write them.
	Be careful before using the write command.

	Command (m for help): t
	Selected partition 1
	Hex code (type L to list all codes): fd
	Changed type of partition 'Linux' to 'Linux raid autodetect'

</details>

Создал RAID1 с одним отсутствующим диском

	mdadm --create /dev/md0 --l 1 -n 2 missing /dev/sdb1
	
<details> <summary>Вывод lsblk:</summary>
	
		NAME    MAJ:MIN RM SIZE RO TYPE  MOUNTPOINT
		sda       8:0    0  40G  0 disk
		`-sda1    8:1    0  40G  0 part  /
		sdb       8:16   0  40G  0 disk
		`-sdb1    8:17   0  40G  0 part
 		 `-md0   9:0    0  40G  0 raid1
</details>

Создал ФС на md0 

	mkfs.ext4 /dev/md0
	
Смонтировал md0 в /mnt и скопировал туда содержимое /

	mount /dev/md0 /mnt/
	rsync -axu / /mnt
	
Смонтировал системные директории в /mnt/ и сделал /mnt/ корнем

	mount --bind /proc /mnt/proc
	mount --bind /dev /mnt/dev
	mount --bind /sys /mnt/sys
	mount --bind /run /mnt/run
	chroot /mnt/
	
Поменял в /etc/fstab uuid диска sda на uuid md0 и файловую систему на ext4

Создал конфиг mdadm

	mdadm --detail --scan > /etc/mdadm.conf
	
Забекапил и создал новый initramfs

	mv /boot/initramfs-3.10.0-957.1.3.el7.x86_64.img /boot/initramfs-3.10.0-957.1.3.el7.x86_64.img.bak
	dracut /boot/initramfs-$(uname -r).img $(uname -r)
	
Добавил в /etc/default/grub опцию "rd.auto=1" для автоопределения и запуска RAID

Создал новый конфиг GRUB и установил GRUB на sdb

	grub2-mkconfig -o /boot/grub2/grub.cfg
	grub2-install /dev/sdb

Далее перезгрузил виртуалку, выбрал другой загрузочный диск, но не смог подключиться по ssh и не смог залогиниться в консоли virtualbox. Выяснил, что дело в selinux. При следующей перезагрузке добавил в конфиг grub опцию "selinux=0" и удачно смог подключиться.

При загрузке, раздел sda1 стал с RAID. Поменял тип раздела sdb на Linux raid autodetect. 

Добавил раздел sdb1 к RAID и записал на него загрузчик.
	
	mdadm --manage /dev/md0 --add /dev/sdb1
	grub2-install /dev/sdb
	
Выключил selinux в файле конфигурации /etc/selinux/config.

Перезагрузился с начального диска.

<details> <summary>Вывод lsblk после перезагрузки:</summary>
	
	NAME    MAJ:MIN RM SIZE RO TYPE  MOUNTPOINT
	sda       8:0    0  40G  0 disk
	`-sda1    8:1    0  40G  0 part
	  `-md0   9:0    0  40G  0 raid1 /
	sdb       8:16   0  40G  0 disk
	`-sdb1    8:17   0  40G  0 part
	  `-md0   9:0    0  40G  0 raid1 /

</details>

P.S. отключать selinux плохо, но я не знаю, как правильно настроить в таком случае...
