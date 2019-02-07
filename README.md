# raid
Задание на **

В вагрант файле добавил второй диск

	:sata1 => {
			:dfile => '/home/otus-linux/sata1.vdi',
			:size => 40960,
			:port => 1

<details> <summary>Вывод lsblk:</summary>
	
		NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
		sda      8:0    0  40G  0 disk
		`-sda1   8:1    0  40G  0 part /
		sdb      8:16   0  40G  0 disk
</details>
