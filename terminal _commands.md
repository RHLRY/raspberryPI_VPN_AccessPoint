https://www.instructables.com/id/Raspberry-Pi-VPN-Gateway/

### Setting up OpenVPN
    mkdir VPNhotspot
    cd VPNhotspot
    unzip openvpn.zip
    wget https://s3.amazonaws.com/tunnelbear/linux/openvpn.zip

## Move these settings files to /etc/openvpn/
    sudo cp CACertificate.crt PrivateKey.key UserCertificate.crt us.conf /etc/openvpn/
    
    cd /etc/openvpn
    sudo nano auth.txt
## Content of auth.txt
    raulrahulroy@gmail.com
    rahulroy@tunnelbear

## rename TunnelBear United States.ovpn to us.conf
Any .ovpn can be used but remember to change to .conf. (remove spaces, as it can create problem)
    
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


    
