### install latest openwrt snapshot on wr710n

ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no root@192.168.1.1

passwd # set password to "root"
opkg remove firewall --force-removal-of-dependent-packages
rm /etc/firewall.user /etc/config/firewall
sed -i 's/192.168.1.1/192.168.2.1/' /etc/config/network
/etc/init.d/network reload

ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no root@192.168.2.1

opkg update
cd ~
opkg install kmod-usb-audio kmod-sound-core nano shairport-sync-openssl alsa-utils --download-only
rm -rf /tmp/opkg-lists
opkg install *
rm *

echo -e "config shairport-sync 'shairport_sync'
	option respawn '1'
	option name 'Master Bedroom'
	option sesctl_session_interruption 'yes' # no/yes
	option alsa_mixer_control_name 'PCM'
" > /etc/config/shairport-sync

/etc/init.d/shairport-sync restart

### audio loopback

echo "#!/bin/sh /etc/rc.common

START=99
STOP=1
USE_PROCD=1

start_service() {
	procd_open_instance
	procd_set_param command nice -n -19 ash -c 'nice -n 10 arecord -D plughw:0 -f dat --disable-softvol | nice -n -19 aplay -f cd --disable-softvol'
	procd_set_param respawn
	procd_close_instance
}
" > /etc/init.d/alsa-loopback

chmod a+x /etc/init.d/alsa-loopback

/etc/init.d/alsa-loopback enable
/etc/init.d/alsa-loopback start

### wireless client mode

root@OpenWrt:/etc/config# cat network

config interface 'loopback'
	option ifname 'lo'
	option proto 'static'
	option ipaddr '127.0.0.1'
	option netmask '255.0.0.0'

config globals 'globals'
	option ula_prefix 'fd6e:135f:0574::/48'

config interface 'lan'
	option type 'bridge'
	option ifname 'eth1'
	option proto 'static'
	option ipaddr '192.168.2.1'
	option netmask '255.255.255.0'
	option ip6assign '60'

config interface 'wan'
	option proto 'dhcp'

config interface 'wan6'
	option proto 'dhcpv6'

root@OpenWrt:/etc/config# cat wireless

config wifi-device 'radio0'
	option type 'mac80211'
	option channel '6'
	option hwmode '11g'
	option path 'platform/ar933x_wmac'
	option htmode 'HT20'

config wifi-iface
	option network 'wan'
	option device 'radio0'
	option ssid 'Blondie'
	option encryption 'psk2'
	option mode 'sta'
	option key 'beachball'


