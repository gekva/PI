This is something I have been using for awhile now, thought i would show you the entire process, This is a tutorial for setting up a Raspberry Pi VPN router.

REQUIREMENTS FOR RASPBERRY PI VPN ROUTER
Raspberry Pi 3 ► Amazon | Ebay

Private Internet Access ► https://goo.gl/StVNEU
Install Raspbian Pixel to your Pi’s sdcard. Use the Raspberry Pi Configuration tool or

sudo raspi-config 
to:

Boot to console
Configure the right keyboard map and timezone
Configure the Memory Split to give 16Mb (the minimum) to the GPU

Install  iptables
sudo apt-get install iptables

Install Speedtest
sudo apt-get install speedtest-cli

Install Cockpit
sudo apt-get install cockpit

STATIC IP ADDRESS
/etc/network/interfaces
like so:

auto lo
iface lo inet loopback

auto eth0
allow-hotplug eth0
iface eth0 inet static
    address 192.168.1.2
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4
SETUP VPN CLIENT
installing openvpn client

sudo apt-get install openvpn
Downloading and uncompressing PIA OpenVPN profiles

wget https://www.privateinternetaccess.com/openvpn/openvpn.zip
unzip openvpn.zip -d openvpn
Copy the profile and certificates to OpenVPN Folder

sudo cp openvpn/ca.rsa.2048.crt openvpn/crl.rsa.2048.pem /etc/openvpn/

sudo cp openvpn/US New York.ovpn /etc/openvpn/US.conf
notice that the extension has changed from ovpn to conf create a login file with username and password for PIA

sudo nano /etc/openvpn/login
add your username and password per line

username1234

password1234
now we need to change the config file to point to correct file locations

sudo nano /etc/openvpn/US.conf
change the following from this:

auth-useriptables-pass

ca ca.rsa.2048.crt

crl-verif crl.rsa.2048.pem
to:

auth-user-pass /etc/openvpn/login

ca /etc/openvpn/ca.rsa.2048.crt

crl-verif /etc/openvpn/crl.rsa.2048.pem
remember to reboot

TESTING THE VPN
before moving forward with forwarding traffic, lets test out the connection

sudo openvpn --config /etc/openvpn/US.conf
to Exit use Ctrl + c Enable VPN at boot

sudo systemctl enable openvpn@US
Setup Forwarding and IPTables (routes)

TO ENABLE FORWARDING
sudo nano /etc/sysctl.conf
uncomment the # to allow forwarding

net.ipv4.ip_forward = 1
you can enable the service by typing this command

sudo sysctl -p
IPTables this is best to just copy and past this to your ssh session. If you want to know more details about these rules, check out the video

sudo iptables -A INPUT -i lo -m comment --comment "loopback" -j ACCEPT
sudo iptables -A OUTPUT -o lo -m comment --comment "loopback" -j ACCEPT
sudo iptables -I INPUT -i eth0 -m comment --comment "In from LAN" -j ACCEPT
sudo iptables -I OUTPUT -o tun+ -m comment --comment "Out to VPN" -j ACCEPT
sudo iptables -A OUTPUT -o eth0 -p udp --dport 1198 -m comment --comment "openvpn" -j ACCEPT
sudo iptables -A OUTPUT -o eth0 -p udp --dport 123 -m comment --comment "ntp" -j ACCEPT
sudo iptables -A OUTPUT -p UDP --dport 67:68 -m comment --comment "dhcp" -j ACCEPT
sudo iptables -A OUTPUT -o eth0 -p udp --dport 53 -m comment --comment "dns" -j ACCEPT
sudo iptables -A FORWARD -i tun+ -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o tun+ -m comment --comment "LAN out to VPN" -j ACCEPT
sudo iptables -t nat -A POSTROUTING -o tun+ -j MASQUERADE
let make sure to keep the rules persistent across reboots

sudo apt-get install iptables-persistent
the installer will ask to save the rules, select YES now if you have new rules you want to add, do

sudo netfilter-persistent save
now lets apply this to startup

sudo systemctl enable netfilter-persistent
ALMOST DONE At this point you can now point your computer gateway to your Raspberry Pi IP address. Now you got a fully functional Raspberry Pi VPN Router. Check  the video for more info 