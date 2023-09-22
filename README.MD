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

