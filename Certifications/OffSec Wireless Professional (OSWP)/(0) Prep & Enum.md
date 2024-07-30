### Setup Monitor Mode
---
```sh
iwconfig
```
- Check if the wireless card is in monitor mode.

###### Kill conflicting processes
```sh
sudo airmon-ng check kill
```

###### Start monitor mode
```sh
sudo airmon-ng start wlan0
```


### Enumerate Networks
---
##### Basic Scan:
###### 2.4GHz with Channel Hopping
```sh
sudo airodump-ng wlan0mon
```
- `BSSID` = Access Point
- `STATION` = Client

##### Specific Channel Scan:
Focus on the channel of the targeted AP to enumerate potential clients:
```sh
sudo airodump-ng -c <channel no.> wlan0mon
```
- Check to see if there are connected clients, and if they are probing any APs.

##### Precise Sniffing:
To focus on particular AP and output as file, do:
```sh
sudo airodump-ng -c <channel no.> --bssid <AP bssid> -w <output file> wlan0mon
```

##### Identify Protocol:
###### WEP:
```sh
sudo airodump-ng wlan0mon --encrypt WEP
```

###### WPA Personal:
The AUTH column will show PSK.

###### WPA Enterprise:
The AUTH column will show MGT. 