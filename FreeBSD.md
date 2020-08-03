[Команды Freebsd](http://vds-admin.ru/unix-commands) - man'ы на русском<br>
[Dualboot Windows и FreeBSD](http://unix1.jinr.ru/~lavr/dual/dualboot.html)<br>
[FreeBSD installed. Your Next Five Moves Should be…](http://twisteddaemon.com/post/92921205276/freebsd-installed-your-next-five-moves-should)
- - -
#### Информация о системе и ядре
`uname -a; uname -mrs`<br>
`freebsd-version`<br>
`getconf LONG_BIT` - битность системы
#### Информация об оборудовании
`pciconf -lv`<br>
`less /var/run/dmesg.boot`
#### Информация о совместимости с linux
`uname -a; sysctl compat.linux | head -2; grep OVERRIDE_LINUX_ /etc/make.conf`
##### Обновить базу locate
`# /etc/periodic/weekly/310.locate` или `# /usr/libexec/locate.updatedb`<br>
#### Установка часового пояса
`# ln -s /usr/share/zoneinfo/Europe/Moscow /etc/localtime`
- - -
#### СЕТЬ
[freebsd.org](https://www.freebsd.org/doc/handbook/network-aggregation.html)
###### WiFi
[unix.stackexchange.com](https://unix.stackexchange.com/a/467381) [freebsd.org](https://www.freebsd.org/doc/en/books/handbook/network-wireless.html)
```
sysctl net.wlan.devices
pciconf -lv
# kldload iwn   # для нужного модуля и прошивки (если уже не в ядре)
# ifconfig wlan0 create wlandev iwn0
# ifconfig wlan0 up scan
ifconfig wlan0 list scan
# wpa_passphrase my_ssid my_ssid_password >> /etc/wpa_supplicant.conf
# vi /etc/rc.conf
  wlans_iwn0="wlan0"
  ifconfig_wlan0="WPA DHCP"
```
или вручную:<br>
`# wpa_supplicant -i wlan0 -c /etc/wpa_supplicant.conf && service netif restart`<br>
Временно подключить сеть по DHCP (для live-cd):<br>
`# dhclient hn0`<br>
- - -
#### РАБОТА С ДИСКАМИ
[linux.cpms.ru](http://linux.cpms.ru/?p=8183) [lists.freebsd.org](https://lists.freebsd.org/pipermail/freebsd-questions/2011-December/236442.html) [rtfm.co.ua](http://rtfm.co.ua/freebsd-gpart-primer-raboty-s-diskami/) [handbook](https://www.freebsd.org/doc/handbook/geom-glabel.html)<br>
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
`tunefs -L метка /dev/5 things you dont know about amazon web servicesda0p1`   # создать/изменить ФС-зависимую метку для не примонтированного раздела da0p1 (в /dev/ФС);<br>
`tunefs -L "" /dev/da0p1`   # очистить ... ;<br>
`glabel label -v метка /dev/da0p1`   # создать/изменить ФС-независимую метку для раздела da0p1 (в /dev/label);<br>
`glabel clear -v da0p1`   # очистить ... ;<br>
`glabel status`   # просмотр всех меток всех разделов; вновь созданные gpt-метки появятся после перезагрузки;<br>
`gpart modify -i1 -l метка da0`   # создать/изменить gpt-метку раздела da0p1 (в /dev/gpt); появляются в /dev/gpt только если: после создания метки система перезагружена; раздел не примонтирован или примонтирован по /dev/gpt/метка (также см. /etc/fstab).<br>
Для создания файловых систем ext2/3/4 нужно установить порт/пакет __e2fsprogs__:<br>
`mke2fs -t ext4 /dev/da0p1`

#### mount
[cyberciti.biz](http://www.cyberciti.biz/faq/mounting-harddisks-in-freebsd-with-mount-command/)<br>
`ls /dev` - файлы устройств, распознанных системой<br>
`egrep 'da[0-9]|cd[0-9]' /var/run/dmesg.boot` или `# camcontrol devlist` - список дисковых накопителей<br>
`mount` или `df -h` - список примонтированных ФС<br>
`mount -u -o rw <mount_point>` - перемонтировать ФС, если она в режиме "только чтение"<br>
`umount <mount_point>` - отмонтировать<br>
`# mkdir /mnt/cd;  mount_cd9660  /dev/cd0 /mnt/cd` или `mount -t cd9660` - примонтировать CD/DVD<br>
`# mkdir /mnt/usb; mount_msdosfs /dev/da0 /mnt/usb` или `mount -t msdosfs` - примонтировать FAT32-флешку<br>
`# mount /mnt/...` - команда монтирования после добавления в __/etc/fstab__:<br>
```
/dev/cd0     /mnt/cd      cd9660     ro,noauto     0 0
/dev/da0     /mnt/usb     msdosfs    rw,noauto     0 0   # для одной подключённой FAT32-флешки
```
###### Монтирование CD/DVD или FAT32-разделов без root-прав
Пользователь должен состоять в группе __operator__: ``pw groupmod operator -m `whoami` ``.<br>
__operator__ - группа, дающая права на перезагрузку/выключение, монтирование CD-ROM/флешки.
```
# sysctl -w vfs.usermount=1       # добавить в /etc/sysctl.conf
# vi /etc/devfs.conf   # для CD-ROM
  own     /dev/cd0     root:operator
  perm    /dev/cd0     0660
# vi /etc/devfs.rules   # для FAT32-флешки
  [system=10]   # или "localrules=5" ???
  add path 'da*' mode 0660 group operator
# vi /etc/rc.conf
  devfs_system_ruleset="system"   # или "localrules" ???
mount_<filesystem> /dev/... ~/mnt/...
```
###### Монтирование ext2/3/4-разделов
[linux.cpms.ru](http://linux.cpms.ru/?p=7875)
```
pkg install fusefs-ext4fuse
# kldload fuse ; echo 'fuse_load="YES"' >> /boot/loader.conf   # если не сделано ранее
# ext4fuse /dev/<ext-partition> <mount_point>
```
###### Монтирование NTFS-разделов
[bsdportal.ru](http://www.bsdportal.ru/viewtopic.php?f=58&amp;t=27153) [forums.freebsd.org](https://forums.freebsd.org/threads/how-to-use-ntfs-3g-as-a-simple-user-not-root.2458/)
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
#### Монтирование Общей папки из Windows-компьютера
[forums.freebsd.org](https://forums.freebsd.org/threads/filesharing-with-freebsd-10-x-as-virtualbox-guest-on-win7-64bit-host.51180/#post-287631) [lissyara.su](http://www.lissyara.su/articles/freebsd/file_system/mount_smbfs/) [netunix.ru](http://www.netunix.ru/index.php/administration/6-mountshare-page)<br>
Используются: Windows 7, кодировка Windows-1251 (CP1251); FreeBSD 10.2, кодировка ru_RU.UTF-8. На Win-компьютере должна быть общая папка с общим доступом.<br>
Подключение вручную (тестирование доступа к общей папке):<br>
`mount_smbfs -I ip_адрес_или_домен -E utf8:cp1251 //<user>@<win-host>/<shared_folder> /mnt`<br>
Здесь `-E utf8:cp1251` - для адекватного отображения кириллических имён файлов.<br>
Если всё в порядке, FreeBSD попросит ввести пароль пользователя Windows. Также возможно потребуется настройка фаерволла Windows на входящие сообщения от FreeBSD.
```
# vi /etc/nsmb.conf   # соблюдая регистр (кроме пароля)
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
##### Проверка записанного образа (контрольная сумма)
[unix.stackexchange.com](https://unix.stackexchange.com/questions/75483/how-to-check-if-the-iso-was-written-to-my-usb-stick-without-errors#comment749548_272821)<br>
`head -c $(stat -f %z the.iso) /dev/sdc | shasum -a 256` - для Linux `stat -c '%s'` и `sha256sum`<br>
`shasum -a 256 -c CHECKSUM-the.iso.SHA256`   # подсчёт и сверка к.с. образа с прилагаемым файлом с уже подсчитанной к.с.
- - -
#### dump/restore
[Handbook](https://www.freebsd.org/doc/ru_RU.KOI8-R/books/faq/disks.html#idp71930832) [lissyara.su](http://www.lissyara.su/?id=2157) [nix-sa.blogspot.com](http://nix-sa.blogspot.com/2011/09/dump-restore.html)<br>
точки монтирования берутся только из __/etc/fstab__ (монтирование вручную не учитывается), можно указывать /dev/...<br>
`dump -0aLf /path/to/dump.dmp <mount_point>` - без сжатия<br>
`dump -0aLf - <mount_point> | gzip > /path/to/dump.dmp.gz` - со сжатием, `-f -` - запись в стандартный вывод<br>
восстановление из сжатого дампа:<br>
`gzip -dc /path/to/dump.dmp.gz | ( cd <mount_point> ; restore -rf - )`<br>
просмотр дампа:<br>
`restore -if /path/to/dump.dmp` - если дамп не сжатый<br>
`gzip -dc /path/to/dump.dmp.gz | restore -if -` - если дамп сжатый<br>
- - -
##### ОБНОВЛЕНИЕ
[taer-naguur.blogspot.ru](http://taer-naguur.blogspot.ru/2015/05/freebsd10x-update-from-source.html) [vniz.net](http://vniz.net/svn.html) [Handbook](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/makeworld.html) [housecomputer.ru](http://housecomputer.ru/os/unix/bsd/freebsd/updating_freebsd.html#.D0.A8.D0.B0.D0.B3_2:_.D0.BF.D1.80.D0.B8.D0.BC.D0.B5.D0.BD.D0.B5.D0.BD.D0.B8.D0.B5_.D0.BE.D0.B1.D0.BD.D0.BE.D0.B2.D0.BB.D0.B5.D0.BD.D0.B8.D0.B9_.D0.B4.D0.BB.D1.8F_.D1.8F.D0.B4.D1.80.D0.B0) [FreeBSD Documentation Project Primer for New Contributors](https://www.freebsd.org/doc/en_US.ISO8859-1/books/fdp-primer/working-copy-choosing-directory.html)<br>
Скачивание и установка обновлений системы безопасности:<br>
`# freebsd-update fetch install` - # не предназначено для веток STABLE и CURRENT!<br>
`pkg install ca_root_nss`<br>
( co = checkout , up = update , sw = switch )<br>
```
# svnlite co https://svn.freebsd.org/base/stable/12/ /usr/src
# svnlite co https://svn.freebsd.org/ports/head/ /usr/ports
# svnlite co https://svn.FreeBSD.org/doc/head /usr/doc
```
[официальные репозитории исходников ядра в дальнейшем](http://blog.tavda.net/2012/09/subversion-freebsd.html) только обновлять:
`# svnlite up /usr/ports`
для переключения на другой репозиторий:
```
# svnlite info /usr/src | grep URL   # узнать адрес текущего репозитория
# svnlite sw https://svn.freebsd.org/base/stable/12 /usr/src
# svnlite up /usr/src   # обновление исходников ядра
```
`cd /usr/src ; less UPDATING` - ознакомиться перед обновлением
```vim /etc/make.conf
  CPUTYPE?=native   # автоматическое определение типа процессора и доступных оптимизаций
```
`# ln -s /root/kernels/MYKERNEL /usr/src/sys/amd64/conf/` - создать симв. ссылку на файл конфигурации собственного ядра, если GENERIC не устраивает
В MYKERNEL удобно использовать:<br>
```
include GENERIC   # дальше описываются лишь различия от GENERIC
ident MYKERNEL
options ...   # или nooptions
device ...   # или nodevice
```
Либо скопировать GENERIC:<br>
`# cd /usr/src/sys/amd64/conf; cp GENERIC MYKERNEL; vi MYKERNEL`<br>
Можно создать файл со всеми возможными опциями ядра (без описания) для справки:<br>
`# cd /usr/src/sys/amd64/conf && make LINT`<br>
```
# chflags -R noschg /usr/obj/* ; rm -rf /usr/obj   # чистка от старых obj-файлов
# cd /usr/src ; make -j2 buildworld   # запуск компиляции базовой системы; -j2 ускоряет процесс (в 2 потока)
# make -j2 buildkernel KERNCONF=MYKERNEL   # запуск компиляции ядра
# make installkernel KERNCONF=MYKERNEL   # установка скомпилированного ядра
# shutdown now   # перезагрузка в однопользовательский режим
# mount -u / ; mount -a -t ufs ; swapon -a   # если система установлена на UFS
# либо
# zfs set readonly=off zroot ; zfs mount -a   # если система установлена на ZFS
# adjkerntz -i   # если системные часы установлены в локальное время
# cp -Rp /etc /etc.bak   # резервная копия /etc
# mergemaster -p   # первоначальное обновление конфигурационных файлов /etc (https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/makeworld.html#mergemaster); для интерактивного слияния файлов, ранее изменённых лично, следует выбирать вариант "m"
# cd /usr/src ; make installworld   # установка скомпилированной базовой системы из /usr/obj
# mergemaster -iFU   # окончательное обновление конфигурационных файлов /etc
# cd /usr/src ; make check-old ; yes|make delete-old   # очистка от старых (потенциально опасных) файлов без запросов на удаление
# reboot
# portmaster -L ; portmaster -a   # обновить все порты (https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/makeworld.html#make-delete-old), либо:
# portmaster --no-confirm -ybda   # обновить все порты без вопросов (http://forum.lissyara.su/viewtopic.php?t=42311)
# cd /usr/src ; yes|make delete-old-libs   # очистка от старых библиотек
```
/usr/src/UPDATING<br>
Перед обновлением до мажорной версии следует сперва обновить текущую.<br>
###### Наиболее безопасный способ сборки ядра:
```
make kernel-toolchain
make -DALWAYS_CHECK_MAKE buildkernel KERNCONF=YOUR_KERNEL_HERE
make -DALWAYS_CHECK_MAKE installkernel KERNCONF=YOUR_KERNEL_HERE
```
###### Проверка наличия новой версии RELENG
`svnlite cat https://svn.freebsd.org/base/releng/12.1/sys/conf/newvers.sh | grep -B2 BRANCH=\"`<br>
Установочные образы: [ftp-archive.freebsd.org](ftp://ftp-archive.freebsd.org/pub/FreeBSD/releases/ISO-IMAGES/)

##### ОБНОВЛЕНИЕ УДАЛЁННО
[FreeBSD 10 удаленное обновление ядра и мира](http://www.fryaha.ru/freebsd-10-update-world-kernel) [Удаленная сборка и установка ядра FreeBSD](http://vds-admin.ru/freebsd/udalennaya-sborka-i-ustanovka-yadra) [Remote FreeBSD server upgrade - Guide! '09](http://daemonforums.org/showthread.php?t=2836)
```
cp -Rp /boot/kernel /boot/kernel.good   # резервная копия текущего (работающего корректно) ядра
cd /usr/src ; make -j2 buildkernel KERNCONF="MYKERNEL GENERIC"   # последовательная сборка нескольких ядер, если нужно
make installkernel KERNCONF=MYKERNEL
make installkernel KODIR=/boot/kernel.generic   # дополнительно: установка ядра GENERIC в отдельный каталог
```
Перед обновлением мира остановить все сервисы кроме sshd и сети (dhcpd):<br>
```
sockstat ; ps aux   # проанализировать вывод команд и выявить запущенные сервисы
service service stop   # cron devd moused ntpd powerd sendmail syslogd nginx ...
cp -Rp /etc /etc.bak   # и так далее…
````
###### Контроль за процессом сборки мира (варианты):
использовать __screen__; вернуться в сессию после восстановления связи с сервером: `screen -dr`
```
make buildworld && mail -s "Buildworld successful!"
make -DNO_CLEAN buildworld   # продолжить процесс сборки, если он был прерван
file /usr/obj/usr/src/usr.sbin/zzz/zzz*   # наличие файла указывает на корректное завершение сборки мира
```
###### Проверка запуска системы с новым ядром
```
cd /usr/src ; make KERNCONF=MYKERNEL buildkernel
make installkernel KERNCONF=MYKERNEL KODIR=/boot/kernel.mykernel
nextboot -k kernel.mykernel   # следующая перезагрузка - с указанным ядром
reboot
mv /boot/kernel /boot/kernel.bak && mv /boot/kernel.mykernel /boot/kernel   # при корректной работе системы
```
В случае неудачи потеряется связь с сервером! Удалённая принудительная перезагрузка запустит систему с прежним (рабочим) ядром.

###### Определение версии установленных ядeр (например /boot/kernel.old)
`strings /boot/kernel.old/kernel | tail`<br>
- - -
#### РАБОТА С ПОРТАМИ/ПАКЕТАМИ
```
/usr/local/etc/pkg/repos/FreeBSD.conf:
FreeBSD: {
  url: "pkg+http://pkg.FreeBSD.org/${ABI}/latest"
}
```
Обновления будут приходить чаще.

###### Ошибка сборки порта. Может возникать при параллельной сборке (make -jX)
`cd /usr/ports/devel/llvm37 ; make MAKE_JOBS_UNSAFE=yes install clean` - или внести `M..=yes` в __make.conf__

###### Поиск пакета с неточно известным названием:
`pkg search -D <pattern>`
###### Вывод информации о еще не установленном пакете
`pkg search -f имя_пакета`

###### Предотвратить переустановку пакета pkg, установленного из портов
[forums.freebsd.org](https://forums.freebsd.org/threads/pkg-and-options-changed.65535/post-384699)<br>
Создать личный локальный репозиторий:<br>
```
# mkdir -p /usr/local/etc/pkg/repos
# vi /usr/local/etc/pkg/repos/local.conf
  local: {
    url: "file:///usr/ports/packages"
    enabled: yes
  }
# mkdir /usr/ports/packages
# make -C /usr/ports/sysutils/fusefs-ntfs package
# pkg repo /usr/ports/packages
# pkg update
# pkg install -r local -f fusefs-ntfs
```
> Thanks to CONSERVATIVE_UPGRADE pkg will now strongly prefer the local repository over others when looking at mailman and pkg shouldn't try to upgrade it from the FreeBSD repo anymore.
#### Удалить все установленные порты и пакеты
[forums.freebsd.org](https://forums.freebsd.org/threads/what-is-the-best-way-to-remove-all-packages-and-start-over.32347/)
Обновить коллекцию портов<br>
```
# pkg delete -a`<br>
# cp /usr/local/etc{,.bak}
# rm -Rf /var/db/pkg/* /usr/local/* /usr/ports/distfiles/*
```
- - -
#### Восстановить консоль после закрытия полноэкранных приложений (less, vi)
[superuser.com](https://superuser.com/a/1248727) [forums.freebsd.org](https://forums.freebsd.org/threads/bash-returning-from-vim.8726/#post-51511)
```
# vi /usr/share/misc/termcap  # или ~/.termcap
  xterm-256color|xterm alias 3:\ :Co#256:pa#32767:\ :AB=\E[48;5;%dm:AF=\E[38;5;%dm:tc=xterm-new:\ :te=\E[2J\E[?47l\E8:ti=\E7\E[?47h::tc=xterm-xfree86:
# cap_mkdb /usr/share/misc/termcap
^D
```
#### Если не работает звук Intel HDA
[opennet.ru](http://www.opennet.ru/openforum/vsluhforumID3/118855.html#80)
Иногда помогает настройка `dev.hdac.0.polling=1` в __/etc/sysctl.conf__.<br>
Далее настроить устройство, куда звук выводится: `hw.snd.default_unit=0` - обычно для вывода на колонки и `=1` для вывода на наушники.<br>
Затем проверить сам звук: `cat /dev/random > /dev/dsp`.
#### Создание dat-файлов для fortune
[скрипт по ссылке не работает](http://bradthemad.org/tech/notes/fortune_makefile.php)<br>
`for i in *; do strfile "$i" "$i.dat"; done`
