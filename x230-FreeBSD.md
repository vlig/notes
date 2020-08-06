#### Видео
`pkg install drm-legacy-kmod gpu-firmware-kmod xf86-video-intel libva-intel-driver` - исправление тиринга [forums.freebsd.org](https://forums.freebsd.org/threads/drm-next-kmod-screen-tearing.66159/#post-390453) [Handbook](https://www.freebsd.org/doc/handbook/x-config.html)
`# vi /usr/local/etc/X11/xorg.conf.d/driver-intel.conf`
```
Section "Device"
        Identifier      "Card0"
        Driver          "intel"
        Option "AccelMethod" "sna"
        Option "TearFree" "true"
EndSection
```
#### Файлы конфигурации
##### /boot/loader.conf
```
autoboot_delay="2"
aesni_load="YES"
geom_eli_load="YES"
kern.geom.label.disk_ident.enable="0"
kern.geom.label.gptid.enable="0"
zfs_load="YES"

iwn6000g2afw_load="YES"
net.inet.tcp.soreceive_stream="1"
net.link.ifqmaxlen="2048"
cc_htcp_load="YES"

acpi_ibm_load="YES"
coretemp_load="YES"
cpuctl_load="YES"
# https://www.c0ffee.net/blog/freebsd-on-a-laptop/
hw.pci.do_power_nodriver="3"
hw.snd.latency="7"
hint.p4tcc.0.disabled="1"
hint.acpi_throttle.0.disabled="1"
hint.ahcich.0.pm_level="5"
hint.ahcich.1.pm_level="5"
hint.ahcich.2.pm_level="5"
hint.ahcich.3.pm_level="5"
hint.ahcich.4.pm_level="5"
hint.ahcich.5.pm_level="5"
drm.i915.enable_rc6="7"
drm.i915.semaphores="1"
drm.i915.intel_iommu_enabled="1"
kern.ipc.shmseg=1024
kern.ips.shmmni=1024
kern.maxproc=100000

hw.psm.synaptics_support="1"
hw.psm.trackpoint_support="1"
ng_bt3c_load="YES"
##mmc_load="YES"
mmcsd_load="YES"
sdhci_load="YES"
atapicam_load="YES"

fuse_load="YES"
cuse_load="YES"
tmpfs_load="YES"
aio_load="YES"
#autofs_load="YES"
libiconv_load="YES"
libmchain_load="YES"
cd9660_iconv_load="YES"
msdosfs_iconv_load="YES"

vboxdrv_load="YES"
```
##### /etc/rc.conf
```
hostname="x230"
wlans_iwn0="wlan0"
ifconfig_wlan0="WPA DHCP powersave"
create_args_wlan0="country RU regdomain NONE"
wpa_supplicant_program="/usr/local/sbin/wpa_supplicant"
background_dhclient="YES"
#sshd_enable="YES"
ntpd_enable="YES"
ntpd_flags="-g"
# Set dumpdev to "AUTO" to enable crash dumps, "NO" to disable
dumpdev="NO"
zfs_enable="YES"
linux_enable="YES"
clear_tmp_enable="YES"
syslogd_flags="-ss"
sendmail_enable="NONE"

powerd_enable="YES"
powerd_flags="-a hiadaptive -b adaptive"
performance_cx_lowest="Cmax"
economy_cx_lowest="Cmax"

gateway_enable="YES"
pf_enable="YES"
pf_rules="/etc/pf.conf"
ipv6_network_interfaces="none"      # Default is auto
ipv6_activate_all_interfaces="NO"   # this is the default
ip6addrctl_enable="NO"              # New way to disable IPv6 support
ip6addrctl_policy="ipv4_prefer"     # Use IPv4 instead of IPv6
ipv6_activate_all_interfaces="NO"   # Do not automatically add IPv6 addresses

kld_list="/boot/modules/drm2.ko /boot/modules/i915kms.ko"
dbus_enable="YES"
hald_enable="YES"
keymap="ru.win"
moused_enable="YES"
moused_flags="-VH"
#autofs_enable="YES"
hcsecd_enable="YES"
sdpd_enable="YES"
bthidd_enable="YES"
cupsd_enable="YES"
webcamd_enable="YES"

slim_enable=yes

#vm_enable="YES"
vm_dir="zfs:zroot/usr/bhyve"
cloned_interfaces="bridge0 tap0"
ifconfig_bridge0="addm wlan0 addm tap0"
#kld_list="nmdm vmm"
vboxnet_enable="YES"
devfs_system_ruleset="system"

sndiod_enable="YES"
```
