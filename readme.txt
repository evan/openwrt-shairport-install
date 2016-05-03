ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no root@192.168.1.1

passwd # set password to "root"

opkg remove firewall --force-removal-of-dependent-packages
rm /etc/firewall.user /etc/config/firewall
sed -i 's/192.168.1.1/192.168.2.1/' /etc/config/network
/etc/init.d/network reload

ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no root@192.168.2.1

opkg update
cd ~
opkg install kmod-usb-audio kmod-sound-core nano shairport-sync-openssl --download-only
rm -rf /tmp/opkg-lists
opkg install *
rm *

echo -e "config shairport-sync 'shairport_sync'
	option respawn '1'
	option name 'Dining Room'
	option sesctl_session_interruption 'yes' # no/yes
	option alsa_mixer_control_name 'PCM'
" > /etc/config/shairport-sync

/etc/init.d/shairport-sync restart


