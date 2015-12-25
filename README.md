## [Ubuntu] OpenVPN TAP Bridge with VLAN

####Mô hình:

<img src="http://i.imgur.com/d6w4khN.png">

#####Yêu cầu:

- Một máy cài đặt Ubuntu với hai card mạng, 1 card bridge và 1 card private.( Ở bài lab này tôi dùng dải VLAN: 20.0.0.0/24)

- 1 máy client có kết nối Internet

#### Cài đặt
#####Server
- Cài đặt các gói cần thiết:

```sh
  appt-get update
  apt-get install openvpn easy-rsa -y
  apt-get install bridge-utils –y
  
```
- Copy và unpack server.conf.gz vào /etc/openvpn
```sh
gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz > /etc/openvpn/server.conf
```
- Tạo thư mục easy-rsa 
```sh 
cp -r /usr/share/easy-rsa/ /etc/openvpn
mkdir /etc/openvpn/easy-rsa/keys
```
- Sửa  /etc/openvpn/easy-rsa/vars

    <img src ="http://i.imgur.com/iZUXt9B.png">
- Tạo CA Certificate và key
```sh 
cd /etc/openvpn/easy-rsa
. ./vars  
./clean-all
./build-ca
```

- Tạo certificate và private key cho server
```sh 
./build-key-server server
```
- Tạo thông số Diffie Hellman
```sh
openssl dhparam -out /etc/openvpn/dh1024.pem 1024
```

- Copy các certificate và key vừa tạo vào /etc/openvpn
```sh
./build-key client1
```
- Sửa file /etc/openvpn/server.conf

  <img src="http://i.imgur.com/tYlH2E0.png">

  <img src="http://i.imgur.com/0YBn42p.png">

  <img src="http://i.imgur.com/APYuXV0.png">
- Cấu hình /etc/sysctl.conf để forward giữa các dải mạng
```sh 
echo 1 > /proc/sys/net/ipv4/ip_forward
```
- Sửa trong file sysctl.conf

  <img src="http://i.imgur.com/RUlygWt.png">
- copy hai file bridge-start và bridge-stop từ /usr/share/doc/openvpn/examples/sample-scripts/ sang /etc/openvpn và đổi thành đuôi .sh
```sh
cp /usr/share/doc/openvpn/examples/sample-scripts/bridge-start /etc/openvpn
cp /usr/share/doc/openvpn/examples/sample-scripts/bridge-stop /etc/openvpn
cd /etc/openvpn
mv bridge-start bridge-start.sh
mv bridge-stop bridge-stop.sh
```
- Sửa file bridge-start.sh
```sh
eth= “eth_vlan”
eth_ip=”ip_vlan_seerver”
eth_netmask=”eth_netmask_vlan”
eth_broadcast=”eth_broadcast_vlan”

```
    <img src="http://i.imgur.com/0s3OH0Q.png">
    
- Chạy file bridge-start.sh và khởi động dịch vụ openvpn trên server
```sh
cd /etc/openvpn/
bash bridge-start.sh
service openvpn start

```
#####Client

- Cài đặt openvpn trên máy client
Copy ba file từ server sang client
```sh
ca.crt
client.crt
client.key
```
- Sửa file client.opvn 
```sh
dev tap0
#dev tun
...
remote ip_server 1194
```
- Click chuột phải vào OpenVPN GUI-> Run as administrator
  <img src="http://i.imgur.com/xeOhYQR.png">
- Chuột phải vào icon OpenVPN Gui trên Task bar và Connect
    <img src="http://i.imgur.com/u5xMlKz.png">
- Kết nối thành công:

  <img src="http://i.imgur.com/zm7jLa9.png">

- Ping được đến server:

  <img src="http://i.imgur.com/SFsEJ7K.png">
