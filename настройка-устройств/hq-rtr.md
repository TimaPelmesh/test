# HQ-RTR

Роутер головного офиса. Сначала адреса и маршрут. GRE, OSPF, NAT и DHCP — только после того, как HQ-RTR пингует `172.16.2.2`.

Всё ниже из режима настройки. В начале сессии:

```
enable
configure terminal
```

## 1. Имя и DNS

Имя устройства, домен и сервер имён HQ-SRV.

```
hostname hq-rtr
ip domain-name au-team.irpo
ip name-server 10.10.100.2
```

## 2. Адреса

`isp` — линк к провайдеру, `vl100` — серверы, `vl200` — клиенты, `vl999` — управление.

```
interface isp
ip address 172.16.1.2/28
exit
interface vl100
ip address 10.10.100.1/27
exit
interface vl200
ip address 10.10.200.1/28
exit
interface vl999
ip address 10.10.30.1/29
exit
```

## 3. Привязка к портам

Без `connect ip interface` интерфейс остаётся down и ничего не пингуется. `ge0` — к ISP, `ge1` — транк на HQ-SW.

```
port ge0
service-instance ge0/isp
encapsulation untagged
connect ip interface isp
exit
exit
port ge1
service-instance ge1/vl100
encapsulation dot1q 100 exact
rewrite pop 1
connect ip interface vl100
exit
service-instance ge1/vl200
encapsulation dot1q 200 exact
rewrite pop 1
connect ip interface vl200
exit
service-instance ge1/vl999
encapsulation dot1q 999 exact
rewrite pop 1
connect ip interface vl999
exit
exit
write memory
show ip interface brief
show service-instance brief
```

## 4. Маршрут в интернет

Шлюз — адрес ISP на этом линке.

```
ip route 0.0.0.0/0 172.16.1.1
write memory
```

Проверка, что второй роутер доступен. Пока нет пинга — не поднимай туннель.

```
ping 172.16.2.2
```

## 5. Учётка администратора

Пароль `P@ssw0rd`.

```
username net_admin
password P@ssw0rd
role admin
exit
write memory
```

## 6. GRE до BR-RTR

Туннель поверх адресов ISP. Source — свой `172.16.1.2`, destination — сосед `172.16.2.2`.

```
interface tunnel.0
ip address 10.10.10.1/30
ip tunnel 172.16.1.2 172.16.2.2 mode gre
exit
write memory
show interface tunnel.0
ping 10.10.10.2
```

## 7. OSPF

Hello только в туннель. Подсети офиса объявляются, но клиентам Hello не уходит. MD5-ключ `1`, пароль `P@ssw0rd` — как на BR-RTR и Ideco.

```
router ospf 1
passive-interface default
no passive-interface tunnel.0
network 10.10.10.0/30 area 0
network 10.10.100.0/27 area 0
network 10.10.200.0/28 area 0
network 10.10.30.0/29 area 0
area 0 authentication message-digest
exit
interface tunnel.0
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 P@ssw0rd
exit
write memory
```

Подожди 30–40 секунд. Сосед один: `10.10.10.2`, состояние Full.

```
show ip ospf neighbor
show ip ospf interface tunnel.0
show ip route ospf
```

## 8. NAT

`outside` — в сторону ISP, `inside` — клиентские интерфейсы. Пул — кому разрешён выход наружу. Пинг с самого роутера в таблицу не попадает: пингует HQ-SRV.

```
interface isp
ip nat outside
exit
interface vl100
ip nat inside
exit
interface vl200
ip nat inside
exit
interface vl999
ip nat inside
exit
ip nat pool HQ 10.10.30.1-10.10.200.14
ip nat source dynamic inside-to-outside pool HQ overload interface isp
write memory
```

На HQ-SRV оставь работать `ping 77.88.8.8`. Здесь смотри таблицу: `10.10.100.2` должен уйти как `172.16.1.2`.

```
show ip nat translations
show running-config | grep nat
```

Если таблица пустая, правило и пул вводятся заново:

```
no ip nat source dynamic inside-to-outside pool HQ overload interface isp
no ip nat pool HQ
ip nat pool HQ 10.10.30.1-10.10.200.14
ip nat source dynamic inside-to-outside pool HQ overload interface isp
write memory
```

## 9. DHCP для HQ-CLI

Пул с `.2`: адрес роутера `.1` клиентам не отдаётся.

```
ip pool VLAN200 10.10.200.2-10.10.200.14
dhcp-server 1
pool VLAN200 1
mask 28
gateway 10.10.200.1
dns 10.10.100.2
domain-name au-team.irpo
exit
exit
interface vl200
dhcp-server 1
exit
write memory
show running-config dhcp-server 1
show dhcp-server clients vl200
```

## 10. Часовой пояс

```
ntp timezone utc+3
write memory
```
