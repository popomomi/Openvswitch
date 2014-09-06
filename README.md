

## LAB KVM - VLAN:

#### I. Mô Hình LAB :
   <img src="http://i.imgur.com/ldl0bbU.png">

#### II. Thông tin thiết bị:
 
1. Máy HOST1:
     - OS: UBUNTU 12.04 DESKTOP
     - HDD: 40GB
     - RAM :1GB
     - CPU : 4 , có tích VT-X
	 
2.Máy HOST2: 
     - OS: UBUNTU 12.04 DESKTOP
     - HDD: 40GB
     - RAM :1GB
     - CPU : 4, có tích VT-X
#### III.Cài Đặt và cấu hình :
##### Cài đặt gói :
1. Cấu hình kiểm tra KVM :
  -  Cài đặt gói:
```
  sudo apt-get install cpu-checker
```
- Lệnh kiểm tra:
```
kvm-ok
```
- Nếu  máy có hỗ trợ ảo hóa sẽ được thông tin :
   
   <img src="http://i.imgur.com/rgcGH6y.png">
2. Cài đặt và cấu hình KVM :
```
sudo apt-get install -y kvm libvirt-bin pm-utils   
```

3.Cài đặt Openvswitch :
```
sudo apt-get install -y openvswitch-switch openvswitch-datapath-dkms
```
#### Cấu hình Openvswitch :
- Tạo một card br0 :
```
 sudo ovs-vsctl add-br br0 
```
- Thêm eth0 vào card br0 :
``` 
  ovs-vsctl add-port br0 eth0
```
- Cấu hình ip cho  br0 :
```
ifconfig eth0 0
ifconfig br0 10.10.10.71 netmask 255.255.255.0
route add default gw 10.10.10.1/24 dev br0
```
- sửa file /etc/network/interface :
```
auto eth0
iface eth0 inet manual
up ifconfig $IFACE 0.0.0.0 up
up ip link set $IFACE promisc on
down ip link set $IFACE promisc off
down ifconfig $IFACE down

auto br0
iface br0 inet static
address 10.10.10.71
netmask 255.255.255.0
gateway 10.10.10.1
dns-nameservers 8.8.8.8
```

4.Tạo máy ảo : 
 
- Trên máy HOST1 :

- Tạo VM1:
```
  kvm -m 512 -net nic,macaddr=12:42:52:CC:CC:15 -net tap  cirros-0.3.2-x86_64-disk.img
```
- nếu cài đặt trên UBUNTU SERVER chạy lênh :
```
 kvm -m 512 -net nic,macaddr=12:42:52:CC:CC:15 -net tap  cirros-0.3.2-x86_64-disk.img -nographic
```
- Tạo VM2 :
```
kvm -m 512 -net nic,macaddr=12:42:52:CC:CC:15 -net tap  cirros-0.3.2-x86_64-disk.img 
```
Sau khi tạo xong VM, ta gắn VM 1, VM 2 vào bridge br0 (vì tham số -net tap trong lệnh trên đã tạo ra một “dây nhẩy mạng”, gắn vào VM.
Clone session mới để truy cập trở lại vào Server vật lý (vì console VM 2 đã chiếm màn hình ssh).

```
ovs-vsctl add-port br0 tap0
ovs-vsctl add-port br0 tap1

```
Trong đó tap0, được tạo ra khi tạo VM1, tap1 tạo ra khi thực hiện lệnh tạo VM2. Thông thường tap sẽ tăng dần tap0, tap1





