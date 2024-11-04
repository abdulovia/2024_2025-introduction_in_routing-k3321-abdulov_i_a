University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)

Year: 2024/2025

Group: K3321

Author: Abdulov Ilia Alex

Lab: Lab1

Date created: 29.09.2024

Date finished: 

## Prerequisites

Перед выполнением лабораторной работы выполним следующие задачи:
- установим [Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- установим пакет *build-essential*: ```sudo apt install build-essential -y```
- склонируем репозиторий *hellt/vrnetlab*: ```git clone https://github.com/hellt/vrnetlab.git```
- установим routeros: ```cd vrnetlab/routeros/; wget https://download.mikrotik.com/routeros/6.47.9/chr-6.47.9.vmdk; make docker-image```
- установим ContainerLab: ```sudo bash -c "$(curl -sL https://get.containerlab.dev)"```
- установим qemu-kvm по инструкции [Installation of KVM](https://help.ubuntu.com/community/KVM/Installation): ```sudo apt-get install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils```, проверим запуск kvm командой *kvm-ok*

## Основная часть лабораторной работы

Напишем [clab yaml](networklab.clab.yaml) и сделаем deploy.

При вызове *clab inspect* должны отображаться созданные контейнеры со статусом: running
![clab inspect](image-inspect.png)

Настроим конфигурации на устройствах:

- R01.TEST

```mikrotik
/interface vlan
add interface=ether2 name=vlan10 vlan-id=10
add interface=ether2 name=vlan20 vlan-id=20
/ip address
add address=10.10.10.2/24 interface=vlan10 
add address=10.10.20.2/24 interface=vlan20
/ip pool
add name=dhcp_pool_vlan10 ranges=10.10.10.25-10.10.10.225
add name=dhcp_pool_vlan20 ranges=10.10.20.25-10.10.20.225
/ip dhcp-server
add address-pool=dhcp_pool_vlan10 disabled=no interface=vlan10 name=dhcp_vlan10
add address-pool=dhcp_pool_vlan20 disabled=no interface=vlan20 name=dhcp_vlan20
/ip dhcp-server network
add address=10.10.10.0/24 gateway=10.10.10.2
add address=10.10.20.0/24 gateway=10.10.20.2
/user add name=root password=root group=full
```

- SW01.L3.01.TEST

```mikrotik
/interface bridge
add name=bridge_vlan10
add name=bridge_vlan20
/interface vlan
add interface=ether2 name=vlan10_ether2 vlan-id=10
add interface=ether3 name=vlan10_ether3 vlan-id=10
add interface=ether2 name=vlan20_ether2 vlan-id=20
add interface=ether4 name=vlan20_ether4 vlan-id=20
/interface bridge port
add bridge=bridge_vlan10 interface=vlan10_ether2
add bridge=bridge_vlan10 interface=vlan10_ether3
add bridge=bridge_vlan20 interface=vlan20_ether2
add bridge=bridge_vlan20 interface=vlan20_ether4
/ip dhcp-client
add disabled=no interface=bridge_vlan10
add disabled=no interface=bridge_vlan20
/user add name=root password=root group=full
```

- SW02.L3.01.TEST

```mikrotik
/interface bridge
add name=bridge_vlan10
/interface vlan
add interface=ether2 name=vlan10_ether2 vlan-id=10
add interface=ether3 name=vlan10_ether3 vlan-id=10
/interface bridge port
add bridge=bridge_vlan10 interface=vlan10_ether2
add bridge=bridge_vlan10 interface=vlan10_ether3
/ip dhcp-client
add disabled=no interface=bridge_vlan10
/user add name=root password=root group=full
```

- SW02.L3.02.TEST

```mikrotik
/interface bridge
add name=bridge_vlan20
/interface vlan
add interface=ether2 name=vlan20_ether2 vlan-id=20
add interface=ether3 name=vlan20_ether3 vlan-id=20
/interface bridge port
add bridge=bridge_vlan20 interface=vlan20_ether2
add bridge=bridge_vlan20 interface=vlan20_ether3
/ip dhcp-client
add disabled=no interface=bridge_vlan20
/user add name=root password=root group=full
```

- PC1

```sh
ip link add link eth1 name vlan10 type vlan id 10 up
udhcpc -i vlan10
```

- PC2

```sh
ip link add link eth1 name vlan20 type vlan id 20 up
udhcpc -i vlan20
```

## Результаты

PC1 и PC2 получают от dhcp сервера на роутере ip-адреса из своей подсети:
![PC2 ip addr is 10.10.20.223](image-dhcp.png)

Компьютеры находятся в разных vlan и intervlan routing на роутере не настроен, поэтому попытки компьютеров PC1 и PC2 пинговать друг друга неуспешны: ![PC1 pings PC2 and R01](image-ping.png)

С роутера ping будет доходить до компьютеров в vlan 10 и 20:
![alt text](image-router.png)

Схема связи классического предприятия:

![main shceme](image-lab1-new.png)

![clab inspect](image-graph.png)
