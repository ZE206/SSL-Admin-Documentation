## Task 7

```bash
sudo apt install wireguard -y
```

### generate keys in wireguard. 
for server keys
```bash
sudo wg genkey | sudo tee /etc/wireguard/server_private.key | wg pubkey | sudo tee /etc/wireguard/server_public.key
```
for client keys
```bash
sudo wg genkey | sudo tee /etc/wireguard/client1_private.key | wg pubkey | sudo tee /etc/wireguard/client1_public.key
sudo wg genkey | sudo tee /etc/wireguard/client2_private.key | wg pubkey | sudo tee /etc/wireguard/client2_public.key
```
### server config

create a server config file - /etc/wireguard/wg0.conf
in here we need to get the address of the server. Using the servers private id, we decrypt the the incoming packets from client and sign the outgoing packets.

Then we allow the packets from VPN interface to be frowarded. When VPN clients apckets goes out to internet interface theat is eth0, replace source IP with servers public IP. NAT->internet sees server's IP not client vpn's ip

The server uses clients public key to verify that packets actually came from this client and to encrypt packets going back to them.

```bash
[Interface]
Address = 10.10.0.1/24
ListenPort = 51820
PrivateKey = oGRQduMVGhkgefpxaNGxlhQNTLSkS0rtXI18ET3nRUE=
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
# Client 1
PublicKey = GOErJsFiTz3cSR/p//jC9bamVEAaswwq3eSjKu9mm2M=
AllowedIPs = 10.10.0.2/32iptables rules that enable NAT masquerading so VPN clients can reach the internet through the server 

[Peer]
# Client 2
PublicKey = dlBjteQf9TCfwTlzGJm8Id3ZGAQscppMKDv9P3Kn3wA=
AllowedIPs = 10.10.0.3/32
```


### IP forwarding
server needs to forwards the data packets between VPN interface (wg0 in thsi case) and internet interface(eth0) . Without it, client connected to VPN would not reach the internet.


### client config


```bash
[Interface]
PrivateKey = IKwkDBMdXVZLGlx7/s99XLpVEmrM4lWXDRYXjQTuQGY=
Address = 10.10.0.2/24
DNS = 8.8.8.8

[Peer]
PublicKey = DtAnBrDIGBEVhsVDpS8T+hGwwtipiNrRKCFqkOeGJHE=
Endpoint = blimnock.sslnitc.site:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

### firewall

add inbound rule in azure with port 51820/UDP in azure 
then 
```bash
sudo ufw allow 51820/udp
```


### last part

start wireguard
```bash
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```



verify it
```bash
sudo wg show
sudo wg show wg0
sudo ufw status
```