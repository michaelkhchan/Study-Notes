### WEP
---
##### wpa_supplicant:
May have to run this on a different wireless interface (i.e., wlan1)

###### Create a `wpa_supplicant.config` file:
```
network={  
	ssid="<name of network>"  
	key_mgmt=NONE  
	wep_key0="<password>"  
	wep_tx_keyidx=0  
}
```

###### Run wpa_supplicant:
```sh
wpa_supplicant -B -D nl80211 -i wlan1 -c wpa_supplicant.config
```
- `-B` for background
- `-D` to indicate the driver (normally nl80211, try cfg80211 if not working)

### WPA Personal
---
###### Create a `wpa_supplicant.config` file:
```
network={
  ssid="<name of network>" 
  scan_ssid=1
  psk="<password>"
  key_mgmt=WPA-PSK
}
```

###### Connect to network:
```sh
sudo wpa_supplicant -i wlan0 -c wpa_supplicant.config
```

If an IP address is need, use DHCP to ask for one.
```sh
sudo dhclient wlan0
```




### WPA Enterprise
---
###### Create a `wpa_supplicant.config` file:
```
network={  
	ssid="<name of network>"  
	scan_ssid=1
	key_mgmt=WPA-EAP
	identity="<Domain\\username>"
	password="<password>"
	eap=PEAP
	phase1="peaplabel=0"
	phase2="auth=MSCHAPV2"
}
```

###### Connect to network:
```sh
sudo wpa_supplicant -i wlan0 -c wpa_supplicant.config
```

If an IP address is need, use DHCP to ask for one.
```sh
sudo dhclient wlan0
```


### Proof.txt
---
```sh
curl http://192.168.1.1/proof.txt
```