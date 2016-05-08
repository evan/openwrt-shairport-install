### install latest openwrt snapshot on wr710n

ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no root@192.168.1.1

passwd # set password to "root"
opkg remove firewall --force-removal-of-dependent-packages
sed -i 's/192.168.1.1/192.168.2.1/' /etc/config/network
/etc/init.d/network reload

ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no root@192.168.2.1

### wireless client mode (optional)

uci delete network.wan.ifname
uci delete network.wan6.ifname
uci delete wireless.radio0.disabled
uci set wireless.@wifi-iface[0].network='wan'
uci set wireless.@wifi-iface[0].ssid='xxx'
uci set wireless.@wifi-iface[0].encryption='psk2'
uci set wireless.@wifi-iface[0].mode='sta'
uci set wireless.@wifi-iface[0].key='xxx'

uci commit

/etc/init.d/network reload

### install shairport

opkg update
for pkg in kmod-usb-audio kmod-sound-core nano shairport-sync-openssl alsa-utils; do
  opkg install "$pkg"
done

echo -e "config shairport-sync 'shairport_sync'
	option respawn '1'
	option name 'Room Name'
	option sesctl_session_interruption 'yes' # no/yes
	option alsa_mixer_control_name 'PCM'
" > /etc/config/shairport-sync

/etc/init.d/shairport-sync restart

### install audio loopback

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
