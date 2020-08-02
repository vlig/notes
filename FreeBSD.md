[Команды Freebsd](http://vds-admin.ru/unix-commands) - man'ы на русском<br>
[Dualboot Windows и FreeBSD](http://unix1.jinr.ru/~lavr/dual/dualboot.html)<br>
[FreeBSD installed. Your Next Five Moves Should be…](http://twisteddaemon.com/post/92921205276/freebsd-installed-your-next-five-moves-should)

#### Восстанавливать консоль после закрытия полноэкранных приложений (less, vi) ([superuser.com](https://superuser.com/a/1248727), [forums.freebsd.org](https://forums.freebsd.org/threads/bash-returning-from-vim.8726/#post-51511))
```
# vim /usr/share/misc/termcap  # или ~/.termcap
  xterm-256color|xterm alias 3:\ :Co#256:pa#32767:\ :AB=\E[48;5;%dm:AF=\E[38;5;%dm:tc=xterm-new:\ :te=\E[2J\E[?47l\E8:ti=\E7\E[?47h::tc=xterm-xfree86:
# cap_mkdb /usr/share/misc/termcap
^D
```

#### [dump](https://www.freebsd.org/cgi/man.cgi?restore%288%29)/[restore](https://www.freebsd.org/cgi/man.cgi?restore%288%29) ([Handbook](https://www.freebsd.org/doc/ru_RU.KOI8-R/books/faq/disks.html#idp71930832), [lissyara.su](http://www.lissyara.su/?id=2157), [nix-sa.blogspot.com](http://nix-sa.blogspot.com/2011/09/dump-restore.html))
точки монтирования берутся только из __/etc/fstab__ (монтирование вручную не учитывается), можно указывать /dev/...<br>
`dump -0aLf /path/to/dump.dmp mount_point` - без сжатия<br>
`dump -0aLf - mount_point | gzip > /path/to/dump.dmp.gz` - со сжатием, `-f -` - запись в стандартный вывод<br>
восстановление из сжатого дампа:<br>
`gzip -dc /path/to/dump.dmp.gz | ( cd /mount_point ; restore -rf - )`<br>
просмотр дампа:<br>
`restore -if /path/to/dump.dmp` - если дамп не сжатый<br>
`gzip -dc /path/to/dump.dmp.gz | restore -if -` - если дамп сжатый<br>

#### СЕТЬ ([freebsd.org](https://www.freebsd.org/doc/handbook/network-aggregation.html))
##### WiFi ([unix.stackexchange.com](https://unix.stackexchange.com/a/467381), [freebsd.org](https://www.freebsd.org/doc/en/books/handbook/network-wireless.html))
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

__operator__ - группа, дающая права на перезагрузку/выключение, монтирование CD-ROM/флешки

