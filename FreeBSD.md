[Команды Freebsd](http://vds-admin.ru/unix-commands) - man'ы на русском<br>
[Dualboot Windows и FreeBSD](http://unix1.jinr.ru/~lavr/dual/dualboot.html)<br>
[FreeBSD installed. Your Next Five Moves Should be…](http://twisteddaemon.com/post/92921205276/freebsd-installed-your-next-five-moves-should)
- - -
#### Восстанавливать консоль после закрытия полноэкранных приложений (less, vi) [superuser.com](https://superuser.com/a/1248727) [forums.freebsd.org](https://forums.freebsd.org/threads/bash-returning-from-vim.8726/#post-51511)
```
# vim /usr/share/misc/termcap  # или ~/.termcap
  xterm-256color|xterm alias 3:\ :Co#256:pa#32767:\ :AB=\E[48;5;%dm:AF=\E[38;5;%dm:tc=xterm-new:\ :te=\E[2J\E[?47l\E8:ti=\E7\E[?47h::tc=xterm-xfree86:
# cap_mkdb /usr/share/misc/termcap
^D
```
- - -
#### dump/restore [Handbook](https://www.freebsd.org/doc/ru_RU.KOI8-R/books/faq/disks.html#idp71930832) [lissyara.su](http://www.lissyara.su/?id=2157) [nix-sa.blogspot.com](http://nix-sa.blogspot.com/2011/09/dump-restore.html)
точки монтирования берутся только из __/etc/fstab__ (монтирование вручную не учитывается), можно указывать /dev/...<br>
`dump -0aLf /path/to/dump.dmp <mount_point>` - без сжатия<br>
`dump -0aLf - <mount_point> | gzip > /path/to/dump.dmp.gz` - со сжатием, `-f -` - запись в стандартный вывод<br>
восстановление из сжатого дампа:<br>
`gzip -dc /path/to/dump.dmp.gz | ( cd <mount_point> ; restore -rf - )`<br>
просмотр дампа:<br>
`restore -if /path/to/dump.dmp` - если дамп не сжатый<br>
`gzip -dc /path/to/dump.dmp.gz | restore -if -` - если дамп сжатый<br>
- - -
#### СЕТЬ [freebsd.org](https://www.freebsd.org/doc/handbook/network-aggregation.html)
###### WiFi [unix.stackexchange.com](https://unix.stackexchange.com/a/467381) [freebsd.org](https://www.freebsd.org/doc/en/books/handbook/network-wireless.html)
```
sysctl net.wlan.devices
pciconv -lv
kldload iwn   # для нужного модуля и прошивки (если уже не в ядре)
ifconfig wlan0 create wlandev iwn0
ifconfig wlan0 up scan
ifconfig wlan0 list scan
# wpa_passphrase my_ssid my_ssid_password >> /etc/wpa_supplicant.conf
sudo -e /etc/rc.conf
  wlans_iwn0="wlan0"
  ifconfig_wlan0="WPA DHCP"
```
или вручную:<br>
`wpa_supplicant -i wlan0 -c /etc/wpa_supplicant.conf && service netif restart`<br>
Временно подключить сеть по DHCP (live-cd):<br>
`# dhclient hn0`<br>
- - -
#### mount [cyberciti.biz](http://www.cyberciti.biz/faq/mounting-harddisks-in-freebsd-with-mount-command/)
`ls /dev` - файлы устройств, распознанных системой
`egrep 'da[0-9]|cd[0-9]' /var/run/dmesg.boot` или `# camcontrol devlist` - список дисковых накопителей
`mount` или `df -h` - список примонтированных ФС
`mount -u -o rw <mount_point>` - перемонтировать ФС, если она в режиме "только чтение"
`umount <mount_point>` - отмонтировать
`# mkdir /mnt/cd;  mount_cd9660  /dev/cd0 /mnt/cd` или `mount -t cd9660` - примонтировать CD/DVD
`# mkdir /mnt/usb; mount_msdosfs /dev/da0 /mnt/usb` или `mount -t msdosfs` - примонтировать FAT32-флешку
`# mount /mnt/...` - команда монтирования после добавления в __/etc/fstab__:
```
/dev/cd0     /mnt/cd      cd9660     ro,noauto     0 0
/dev/da0     /mnt/usb     msdosfs    rw,noauto     0 0   # для одной подключённой FAT32-флешки
```
###### Монтирование CD/DVD или FAT32-разделов без root-прав
Пользователь должен состоять в группе __operator__: ``pw groupmod operator -m `whoami` ``.
__operator__ - группа, дающая права на перезагрузку/выключение, монтирование CD-ROM/флешки.
```
# sysctl -w vfs.usermount=1       # добавить в /etc/sysctl.conf
# vim /etc/devfs.conf   # для CD-ROM
  own     /dev/cd0     root:operator
  perm    /dev/cd0     0660
# vim /etc/devfs.rules   # для FAT32-флешки
  [system=10]   # или "localrules=5" ???
  add path 'da*' mode 0660 group operator
# vim /etc/rc.conf
  devfs_system_ruleset="system"   # или "localrules" ???
mount_<filesystem> /dev/... ~/mnt/...
```
###### Монтирование ext2/3/4-разделов [linux.cpms.ru](http://linux.cpms.ru/?p=7875)
```
pkg install fusefs-ext4fuse
# kldload fuse ; echo 'fuse_load="YES"' >> /boot/loader.conf   # если не сделано ранее
# ext4fuse /dev/<ext-partition> <mount_point>
```
###### Монтирование NTFS-разделов [bsdportal.ru](http://www.bsdportal.ru/viewtopic.php?f=58&amp;t=27153) [forums.freebsd.org](https://forums.freebsd.org/threads/how-to-use-ntfs-3g-as-a-simple-user-not-root.2458/)
```
pkg install fusefs-ntfs
echo 'fuse_load="YES"' >> /boot/loader.conf
echo 'fusefs_enable="YES"' >> /etc/rc.conf
# /usr/local/etc/rc.d/fusefs start
# kldload fuse
# ntfs-3g /dev/<ntfs-partition> <mount_point>
```
###### Монтирование NTFS-разделов без root-прав (спорно!)
Установить SUID-флаг на ntfs-3g: [ntfs-3g suid-flag](https://www.google.ru/search?q=ntfs-3g+suid-flag&amp;oq=ntfs-3g+suid-flag&amp;gs_l=serp.3...2353.11277.0.12038.18.12.3.0.0.0.144.922.10j1.11.0....0...1c.1.64.serp..6.12.713.Yj8H7v3KbjA) [tuxera.com](http://www.tuxera.com/community/ntfs-3g-faq/#useroption), [gentoo-wiki.info](http://www.gentoo-wiki.info/NTFS-3G#ntfs-3g_with_suid_bit)
- - -
#### Установка часового пояса
`# ln -s /usr/share/zoneinfo/Europe/Moscow /etc/localtime`
#### Создание dat-файлов для fortune [скрипт не работает](http://bradthemad.org/tech/notes/fortune_makefile.php)
`for i in *; do strfile "$i" "$i.dat"; done`
- - -
#### Монтирование Общей папки из Windows-компьютера [forums.freebsd.org](https://forums.freebsd.org/threads/filesharing-with-freebsd-10-x-as-virtualbox-guest-on-win7-64bit-host.51180/#post-287631) [lissyara.su](http://www.lissyara.su/articles/freebsd/file_system/mount_smbfs/) [netunix.ru](http://www.netunix.ru/index.php/administration/6-mountshare-page)
Используются: Windows 7, кодировка Windows-1251 (CP1251); FreeBSD 10.2, кодировка ru_RU.UTF-8. На Win-компьютере должна быть общая папка с общим доступом.<br>
Подключение вручную (тестирование доступа к общей папке):<br>
`mount_smbfs -I ip_адрес_или_домен -E utf8:cp1251 //<user>@<win-host>/<shared_folder> /mnt`<br>
Здесь `-E utf8:cp1251` - для адекватного отображения кириллических имён файлов.<br>
Если всё в порядке, FreeBSD попросит ввести пароль пользователя Windows. Также возможно потребуется настройка фаерволла Windows на входящие сообщения от FreeBSD.
```
# vim /etc/nsmb.conf   # соблюдая регистр (кроме пароля)
   [default]
   [WIN_HOST]
   addr=<IP_ADDRESS_OR_DOMAIN>
   [<WIN-HOST>:<WIN-USER>]
   charsets=utf8:cp1251
   password=<encrypted_win_password>
```
Значение параметра `encrypted_win_password` можно получить следующей командой:<br>
` smbutil crypt <unencrypted_password>` - использование пробела перед командой исключает её из истории<br>
Теперь общая папка монтируется проще (по сравнению с командой из п.3), и без запроса пароля:<br>
`mount_smbfs //<win-user>@<win-host>/<shared_folder> /mnt`<br>
Для автоматического монтирования общей папки при загрузке системы, в файл __/etc/fstab__ следует добавить строку:<br>
`//<win-user>@<win-host>/<shared_folder>   /mnt   smbfs   rw,-N,-f660   0   0`
- - -
#### Работа с дисками [linux.cpms.ru](http://linux.cpms.ru/?p=8183) [lists.freebsd.org](https://lists.freebsd.org/pipermail/freebsd-questions/2011-December/236442.html) [rtfm.co.ua](http://rtfm.co.ua/freebsd-gpart-primer-raboty-s-diskami/) [handbook](https://www.freebsd.org/doc/handbook/geom-glabel.html)
`sysctl kern.disks`   # список подключенных дисков<br>
`geom disk list`
`ls /dev/da*`   # список подключённых USB-дисков<br>
`gpart show`   # список размеченных дисков<br>
`gpart show -p ada5`   # список разделов диска ada5; `-p` - отобразить имена устройств, вместо индексов разделов<br>
`gpart list`   # подробная информация о дисках и разделах<br>
`gpart destroy -F ada5`   # удалить сущ. таблицу разделов (не только GPT); `-F` используется, если без него ошибка<br>
`gpart delete -i 3 ada5`   # удалить раздел на диске ada5; `-i` - индекс раздела<br>
`gpart create -s gpt ada5`   #1. Создать схему (геометрию) GPT на диске<br>
`gpart bootcode -b /boot/pmbr ada5`   #2. Создать таблицу разделов формата GPT на диске ada5<br>
`gpart add -t freebsd-swap -a 8 -s 4gb ada5`   #3. Создать swap-раздел, `-s` - размером 4Гб , `-a 8` - выравнивание для дисков Advanced Format в режиме эмуляции 512 байтных секторов (используется в дисках WD)<br>
`gpart add -t freebsd-ufs -l var -a 8 -s 10gb ada5`   #4. (см. #3) UFS-раздел, c меткой var, 10Гб<br>
`gpart add -t freebsd-ufs -a 8 ada5`   #5. (см. #3) UFS-раздел, на всё доступное пространство<br>
`newfs -j -L метка ada5p1`   6. Создать новую ФС / форматирование раздела; `-U` - включить Soft Updates; `-j` - включить журналирование Soft Updates (FreeBSD 9 и выше); `-L` - метка тома, позволяет монтировать ФС из /dev/ufs/метка, без привязки к имени контроллера и номеру порта<br>
`tunefs -p /dev/da0p1`   # информация о файловой системе;<br>
`tunefs -L метка /dev/da0p1`   # создать/изменить ФС-зависимую метку для не примонтированного раздела da0p1 (в /dev/ФС);<br>
`tunefs -L "" /dev/da0p1`   # очистить ... ;<br>
`glabel label -v метка /dev/da0p1`   # создать/изменить ФС-независимую метку для раздела da0p1 (в /dev/label);<br>
`glabel clear -v da0p1`   # очистить ... ;<br>
`glabel status`   # просмотр всех меток всех разделов; вновь созданные gpt-метки появятся после перезагрузки;<br>
`gpart modify -i1 -l метка da0`   # создать/изменить gpt-метку раздела da0p1 (в /dev/gpt); появляются в /dev/gpt только если:<br>
1) после создания метки система перезагружена; 2) раздел не примонтирован или примонтирован по /dev/gpt/метка (также см. /etc/fstab).<br>
Для создания файловых систем ext2/3/4 нужно установить порт/пакет __e2fsprogs__ (см. man):<br>
`mke2fs -t ext4 /dev/da0p1`
- - -
