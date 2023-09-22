1- Updating server.

```
apt-get update && apt-get upgrade -y
```
2- Install necessary dependencies.

```
apt-get install build-essential && apt-get install net-tools
```
3- Download the latest version of SoftEther.

```
wget https://github.com/SoftEtherVPN/SoftEtherVPN_Stable/releases/download/v4.41-9782-beta/softether-vpnserver-v4.41-9782-beta-2022.11.17-linux-x64-64bit.tar.gz
```
4- Extract the installer.

```
tar -xvzf https://github.com/SoftEtherVPN/SoftEtherVPN_Stable/releases/download/v4.41-9782-beta/softether-vpnserver-v4.41-9782-beta-2022.11.17-linux-x64-64bit.tar.gz
```
5- Navigate to new directory and install SoftEther.

```
cd vpnserver
make
```
6- Go back to your home directory, move the extracted directory to /usr/local directory, navigate to the new vpnserver location, and set the permissions of it.

```
cd ..
mv vpnserver /usr/local
cd /usr/local/vpnserver/
chmod 600 *
chmod 700 vpncmd
chmod 700 vpnserver
```
7- Perform a final check to see whether VPN Server can operate properly on your computer system before starting vpnserver. Then exit the test.

```
./vpncmd
```
```
3
```
```
check
```
```
exit
```

8- Create a systemd service file to manage the SoftEther VPN service.

```
nano /etc/init.d/vpnserver
```
```
#!/bin/sh
### BEGIN INIT INFO
# Provides: myscript
# Required-Start:
# Required Stop:
# Default-Start: 2 3 4 5
# Default Stop: 1 0 6
# Short-Description: simple description.
### END INIT INFO
# chkconfig: 2345 99 01
# description: SoftEther VPN Server
DAEMON=/usr/local/vpnserver/vpnserver
LOCK=/var/lock/subsys/vpnserver
test -x $DAEMON || exit 0
case "$1" in
start)
$DAEMON start
touch $LOCK
;;
stop)
$DAEMON stop
rm $LOCK
;;
restart)
$DAEMON stop
sleep 3
$DAEMON start
;;
*)
echo "Usage: $0 {start|stop|restart}"
exit 1
esac
exit 0
```
9- Set proper permission to the systemd service file.

```
chmod 755 /etc/init.d/vpnserver
```
10- Add script to system startup.

```
update-rc.d vpnserver defaults
```
11- Start VPN service.

```
/etc/init.d/vpnserver start
```
12- On the main page of the server manager, press “Local Bridge Setting”. From the Virtual Hub drop down, select yours. Click “Bridge with New Tap Device” and type “soft”. Click “Create Local Bridge”. A prompt may come up for “Using Local Bridge Function on VM” but just press ok. You should see the new device in the list as “Operating”.

13- Go back to the Linux machine using PuTTY. Show to the new “soft” network interface we created.

```
ifconfig tap_soft
```
14- List open processes with IP sockets and do not resolve hostnames or port names.

```
lsof -i -P -n | grep LIST
```
15- systemd-resolved stops port 53 from being used by dnsmasq, which it needs. Stop the systemd-resolved service and make a change to it’s configuration.

```
systemctl stop systemd-resolved
nano /etc/systemd/resolved.conf
```
16- Add this to the end of the config file.

```
DNSStubListener=no
```
17- Start the service.

```
systemctl start systemd-resolved
```
18- Install dnsmasq for DCHP and DNS.

```
apt-get install dnsmasq
```
```
nano /etc/dnsmasq.conf
```
19- Add this to the end of the config file. Press “Ctrl + / ” then “Ctrl + V” to go to the end quickly.

```
interface=tap_soft
dhcp-range=tap_soft,192.168.7.50,192.168.7.60,12h
dhcp-option=tap_soft,3,192.168.7.1
server=1.1.1.1
```

20- Edit the SoftEther VPN service.

```
nano /etc/init.d/vpnserver
```

21- Remove the old config and copy this one.

```
#!/bin/sh
# BEGIN INIT INFO
# Provides: vpnserver
# Required-Start: $remote_fs $syslog
# Required-Stop: $remote_fs $syslog
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: Start daemon at boot time
# Description: Enable Softether by daemon.
# END INIT INFO
DAEMON=/usr/local/vpnserver/vpnserver
LOCK=/var/lock/subsys/vpnserver
TAP_ADDR=192.168.7.1

test -x $DAEMON || exit 0
case "$1" in
start)
$DAEMON start
touch $LOCK
sleep 1
/sbin/ifconfig tap_soft $TAP_ADDR
;;
stop)
$DAEMON stop
rm $LOCK
;;
restart)
$DAEMON stop
sleep 3
$DAEMON start
sleep 1
/sbin/ifconfig tap_soft $TAP_ADDR
;;
*)
echo "Usage: $0 {start|stop|restart}"
exit 1
esac
exit 0
```
22- Update the startup script.

```
update-rc.d vpnserver defaults
```
23- Create the file “/etc/sysctl.d/ipv4_forwarding.conf” and add the following. This sends a network packet from one network interface to another one on the same device.

```
nano /etc/sysctl.d/ipv4_forwarding.conf
```
```
net.ipv4.ip_forward = 1
```

24- Apply these changes.

```
sysctl --system
```
25- Add rule to the firewall. Basically this is saying that anything coming in with that 192.168.7.X address (which your computer/smartphone will get once you connect to the VPN) to forward it out through the VPS’s normal internet connection. Add your own IP address between the brackets.

```
iptables -t nat -A POSTROUTING -s 192.168.7.0/24 -j SNAT --to-source [YOUR VPS IP ADDRESS]
```
26- Install iptables-persistent so that these firewall changes aren’t lost after reboot.

```
apt-get install iptables-persistent
```

27- Restart services.
```
/etc/init.d/vpnserver restart
/etc/init.d/dnsmasq restart
```

28- Typing in this list command should show dnsmasq using port 53.

```
lsof -i -P -n | grep LIST
```
29- Save currently-loaded iptables rules to persistent storage.

```
service netfilter-persistent save
```





