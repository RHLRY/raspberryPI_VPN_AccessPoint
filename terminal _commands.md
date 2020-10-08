https://www.instructables.com/id/Raspberry-Pi-VPN-Gateway/
https://www.raspberrypi.org/documentation/configuration/wireless/access-point-routed.md

### Setting up OpenVPN
    mkdir VPNhotspot
    cd VPNhotspot
    wget https://s3.amazonaws.com/tunnelbear/linux/openvpn.zip
    unzip openvpn.zip
    sudo apt-get install openvpn
    sudo systemctl enable openvpn


## rename TunnelBear United States.ovpn to us.conf
Any .ovpn can be used but remember to change to .conf. (remove spaces, as it can create problem)
    
    cp ~/VPNhotspot/openvpn/'TunnelBear United States.ovpn' ~/VPNhotspot/openvpn/us.conf
    
## Move these settings files to /etc/openvpn/
    sudo cp ~/VPNhotspot/openvpn/CACertificate.crt ~/VPNhotspot/openvpn/PrivateKey.key ~/VPNhotspot/openvpn/UserCertificate.crt ~/VPNhotspot/openvpn/us.conf /etc/openvpn/
    cd /etc/openvpn
    
## edit conf file
    sudo nano us.conf
    
## content of us.conf (Make sure to provide absolute path)
    client
    dev tun0
    proto udp
    comp-lzo
    nobind
    ns-cert-type server
    persist-key
    persist-tun
    reneg-sec 0
    dhcp-option DNS 8.8.8.8
    dhcp-option DNS 8.8.4.4
    redirect-gateway
    verb 1
    auth-user-pass /etc/openvpn/auth.txt
    ca /etc/openvpn/CACertificate.crt
    cert /etc/openvpn/UserCertificate.crt
    key /etc/openvpn/PrivateKey.key
    remote us.lazerpenguin.com 443
    cipher AES-256-CBC
    auth SHA256
    keysize 256    
    
## Make authentication file
    sudo nano auth.txt
    sudo chmod 600 /etc/openvpn/auth.txt

## Content of auth.txt
    raulrahulroy@gmail.com
    rahulroy@tunnelbear
    
## Reboot 
    sudo reboot now
    
    
## tun0 -> $ifconfig -> output
    tun0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
        inet 172.18.13.94  netmask 255.255.255.255  destination 172.18.13.93
        inet6 fde4:8dba:82e3::1016  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::c8a2:bbad:ef4f:75e3  prefixlen 64  scopeid 0x20<link>
        unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  txqueuelen 100  (UNSPEC)
        RX packets 18639  bytes 15522999 (14.8 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 14683  bytes 2451756 (2.3 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    
## Setting up hotspot
    sudo apt install hostapd
    sudo systemctl unmask hostapd
    sudo systemctl enable hostapd
    sudo apt install dnsmasq
    sudo nano /etc/dhcpcd.conf
            interface wlan0
            static ip_address=192.168.4.1/24
            nohook wpa_supplicant
    sudo nano /etc/sysctl.d/routed-ap.conf
            # https://www.raspberrypi.org/documentation/configuration/wireless/access-point-routed.md
            # Enable IPv4 routing
            net.ipv4.ip_forward=1
    sudo iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE
    sudo netfilter-persistent save
    sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
    sudo nano /etc/dnsmasq.conf
            interface=wlan0 # Listening interface
            dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h
                            # Pool of IP addresses served via DHCP
            domain=wlan     # Local wireless DNS domain
            address=/gw.wlan/192.168.4.1
                            # Alias for this router
    sudo rfkill unblock wlan
    sudo nano /etc/hostapd/hostapd.conf
            country_code=IN
            interface=wlan0
            ssid=ROY-VPNsecured
            hw_mode=g
            channel=7
            macaddr_acl=0
            auth_algs=1
            ignore_broadcast_ssid=0
            wpa=2
            wpa_passphrase=!@#$5678
            wpa_key_mgmt=WPA-PSK
            wpa_pairwise=TKIP
            rsn_pairwise=CCMP
    sudo systemctl reboot






    
