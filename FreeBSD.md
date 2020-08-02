[Команды Freebsd](http://vds-admin.ru/unix-commands) - man'ы на русском<br>
[Dualboot Windows и FreeBSD](http://unix1.jinr.ru/~lavr/dual/dualboot.html)<br>
Графика в консоли ([lissyara.su](http://www.lissyara.su/articles/freebsd/trivia/graphical_console/),<br>

<ins>Восстанавливать консоль после закрытия полноэкранных приложений</ins> (less, vi) ([superuser.com](https://superuser.com/a/1248727),
[forums.freebsd.org](https://forums.freebsd.org/threads/bash-returning-from-vim.8726/#post-51511))<br>
Добавить в файл __/usr/share/misc/termcap__ (затем __cap_mkdb__ над ним) или в __~/.termcap__:<br>
`xterm-256color|xterm alias 3:\ :Co#256:pa#32767:\ :AB=\E[48;5;%dm:AF=\E[38;5;%dm:tc=xterm-new:\ :te=\E[2J\E[?47l\E8:ti=\E7\E[?47h::tc=xterm-xfree86:`<br>
Перелогиниться.

[dump](https://www.freebsd.org/cgi/man.cgi?restore%288%29)/[restore](https://www.freebsd.org/cgi/man.cgi?restore%288%29) (ещё: [Handbook](https://www.freebsd.org/doc/ru_RU.KOI8-R/books/faq/disks.html#idp71930832), [lissyara.su](http://www.lissyara.su/?id=2157), [nix-sa.blogspot.com](http://nix-sa.blogspot.com/2011/09/dump-restore.html))<br>
точки монтирования берутся только из __/etc/fstab__ (монтирование вручную не учитывается), можно указывать /dev/...<br>
`dump -0aLf /где/сохранить/дамп.dmp точка_монтирования`<br>
или со сжатием (`-f -` - запись в стандартный вывод):<br>
`dump -0aLf - точка_монтирования | gzip > /где/сохранить/дамп.dmp.gz`<br>
восстановление из сжатого дампа:<br>
`gzip -dc /путь/к/дампу.dmp.gz | ( cd /точка_монтирования ; restore -rf - )`<br>
просмотр дампа:<br>
`restore -if /путь/к/дампу.dmp` - дамп не сжатый<br>
`gzip -dc /путь/к/дампу.dmp.gz | restore -if -` - дамп сжатый<br>

<ins>СЕТЬ</ins> ([freebsd.org](https://www.freebsd.org/doc/handbook/network-aggregation.html))<br>
__WiFi__ ([unix.stackexchange.com](https://unix.stackexchange.com/a/467381), [freebsd.org](https://www.freebsd.org/doc/en/books/handbook/network-wireless.html))
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
или вручную<br>
`wpa_supplicant -i wlan0 -c /etc/wpa_supplicant.conf; service netif restart`
