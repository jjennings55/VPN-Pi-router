# VPN-Pi-router
How to install openWRT onto a Raspberry pi and then add Proton VPN


This project relied heavily on NetworkChuck in this video https://www.youtube.com/watch?v=jlHWnKVpygw  

If you travel a lot, you know there's ways to find internet, but many times it's open wifi.  This pi router allows me to connect to it while being secured behind a VPN.  

Items Needed
1. Raspberry Pi 4 (8gb preferred)
2. Micro SD card (32mb or larger)
3. Micro SD card reader
4. USB wireless adapter

I used OpenWRT since it's free and open sourced.  Go to their website https://openwrt.org/toh/views/toh_fwdownload and download the file you need for your pi.  Download then use Pi Imager to write the download to your micro sd card. For choosing OS, choose custom, then pick the download file.  Choose storage which your sd card should come up in the options.  Click write and wait until finished.

![image](https://user-images.githubusercontent.com/88257942/235709026-7958b9c3-efcf-444b-9fd6-ecf629cc6428.png)


Plug in your programmed sd card into your Pi.  Connect it to your computer via ethernet and power the pi up.  From there you'll need to pull up your control panel and change the properties of your Internet protocal v4 to a static address.  This is because all Pi's come with the default Ip address of 192.168.1.1.

![image](https://user-images.githubusercontent.com/88257942/235708524-58e401fc-2e04-46e4-8b22-3821d96069f6.png)

SSH into the pi from a command prompt in your Windows machine.  The command will be 'ssh root@192.168.1.1' then type yes.  At this point you have a secure shell into the pi, and this would be a good time to change the password.  Type 'passwd' and then type your new password for the root user.

Once in remember the commands will be for linux.  Navigate to the /etc/config directory with the command 'cd /etc/config' then type 'ls" to view files.  Here's a good time to copy your files.  I saved wireless, network, and firewall. To back up use the commands 'cp firewall firewall.backup'.  Rinse and repeat for wireless and network.

Now edit the network file.  'nano network' The screenshots below show original then how they should be set up. 'Ctrl+v then y' to save the file
![image](https://user-images.githubusercontent.com/88257942/235712853-55d02616-3f88-4c32-920d-0e20b547287f.png)

In image below you'll see to change ip address in config interface and add new config interface for wwan.  It's good practice to change the IP cause all bad actors know the private ip address for Pi's out of the box.
![image](https://user-images.githubusercontent.com/88257942/235713961-b5d12532-176a-40b0-b450-92988572757f.png)

Next edit the firewall file.  In config zone go down to config zone and change option input from REJECT to ACCEPT and Save
![image](https://user-images.githubusercontent.com/88257942/235714673-4bcda6d4-795d-41b4-8ef5-2fab6a120c7d.png)

Next reboot your pi by entering 'reboot'.  Before ssh backing in go back to your ethernet IPv4 properties back to automatic DHCP.  This is done from your control panel.

SSh back into your pi.  Remember you have a new IP address and password if you changed it.  
Now you have to connect your pi to your home wifi or whatever wifi you want to use.  

Navigate back to your wireless file in your /etc/config directory.  Add the following changes to your wireless file.  
![image](https://user-images.githubusercontent.com/88257942/235716346-3f671594-831d-4596-9a75-bb87b742569f.png)


Now we need to apply the config changes.  Enter commands 'uci commit wireless' then 'wifi'.  After this OpenWRT should be broadcasting from your pi.  Check with your phone.  

Now we can move to the GUI, and many things can be done from the GUI.  Open a browser and enter the ip address of your pi.  Enter root and password.  Go to the wireless tab and connect your home wifi to your pi.  Save and apply your changes.  

Confirm you have internet on your pi by pinging a server. 

Now time to get the wireless usb adapter working.
Back inside your terminal type 'opkg update' and make sure it went through.  You may need to reboot before applying the update.

After successful update you'll need to install several drivers.  Type 'opkg install kmod-rt2800-lib kmod-rt2800-usb kmod-rt2x00-lib kmod-rt2x00-usb kmod-usb-core kmod-usb-uhci kmod-usb-ohci kmod-usb2 usbutils openvpn-openssl luci-app-openvpn nano'

Now you can plug in your USB wireless adapter.  You can confirm this by typing 'ls usb'.  Then type 'ifconfig wlan1 up'.  If this works your adapter is working correctly.  Review your wireless config file if it doesn't work or you need to change ssid or password.



Network Chuck OpenWRT VPN Raspberry Pi Router commands
Commands needed to establish VPN through Nord or other VPN supplier

# Install packages
opkg update
opkginstall luci-app-openvpn
/etc/init.d/rpcd restart
'''

...
#Configuration parameters
OVPN_DIR="/etc/openvpn"
OPVN_ID="client"
OPVN_USER="USERNAME"
OPVN_PASS="PASSWORD"

# Save username/password creditails
umask go=
cat << EOF >${OVPN_DIR}/${OVPN_ID}.auth
${OVPN_USER}
${OVPN_PASS}
EOF

# Configure VPN service
sed -i -e "
/^auth-user-pass/s/^/#/
\$a auth-user-pass${OVPN_ID}.auth
/^redirect-gateway/s/^/#/
\$a redirect-gateway def1 ipv6
"${OVPN_DIR}/${OVPN_ID}.conf
/etc/init.d/openvpn restart
'''

'''
# Provide VPN instance management
ls /etc/openvpn/*.conf \
| while read -r OVPN_CONF
do
OVPN_ID="$(basename ${OVPN_CONF%.*} | sed -e "s/\W/_/g")"
uci -q delete openvpn.${OVPN_ID}
uci set openvpn.${OVPN_ID}="openvpn"
uci set openvpn.${OVPN_ID}.enabled="1"
uci set openvpn.${OVPN_ID}.config="${OVPN_CONF}"
done
ucie commit openvpn
/etc/init.d/openvpn restart
'''

'''
# Configure firewall
uci rename firewall.@zone[0]="lan"
uci rename firewall.@zone[1]="wan"
uci del_list firewall.wan.device="tun+"
uci add_list firewall.wan.device="tun+"
uci commit firewall
/etc/init.d/firewall restart

If the above commands don't work I found ProtonVPN to be very helpful.  NetworkChuck uses Nord, but I used Proton.  This page walks through how to establish your vpn server with the gui.  Log into your router through your browser and follow steps in link below.
https://protonvpn.com/support/how-to-set-up-protonvpn-on-openwrt-routers/


Thank you for visiting this repository.  I hope it's helpful and useful.  Please let me know if I've missed something here.








