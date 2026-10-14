# BR-RTR

Роутер филиала. Адреса и маршрут — сразу. GRE и OSPF — после пинга `172.16.1.2`. NAT — когда BR-SRV уже имеет адрес.

В начале сессии:

```
enable
configure terminal
```

## 1. Имя и DNS

```
hostname br-rtr
ip domain-name au-team.irpo
ip name-server 10.10.100.2
```

## 2. Адреса

`isp` — к провайдеру, `fw` — к Ideco. Подсеть линка к Ideco сверь с его консолью: здесь `10.20.10.0/30`.

```
interface isp
ip address 172.16.2.2/28
exit
interface fw
ip address 10.20.10.1/30
exit
```

## 3. Привязка к портам

`ge0` — к ISP, `ge1` — к BR-FW. Оба кадра без тега.

```
port ge0
service-instance ge0/isp
encapsulation untagged
connect ip interface isp
exit
exit
port ge1
service-instance ge1/fw
encapsulation untagged
connect ip interface fw
exit
exit
write memory
show ip interface brief
```

## 4. Маршрут в интернет

```
ip route 0.0.0.0/0 172.16.2.1
write memory
ping 172.16.1.2
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

## 6. GRE до HQ-RTR

Зеркало туннеля головного офиса: source `172.16.2.2`, destination `172.16.1.2`.

```
interface tunnel.0
ip address 10.10.10.2/30
ip tunnel 172.16.2.2 172.16.1.2 mode gre
exit
write memory
show interface tunnel.0
ping 10.10.10.1
```

## 7. OSPF

Hello в туннель и в линк на Ideco. Объявляются только эти две подсети: сети за Ideco объявляет сам BR-FW.

```
router ospf 1
passive-interface default
no passive-interface tunnel.0
no passive-interface fw
network 10.10.10.0/30 area 0
network 10.20.10.0/30 area 0
area 0 authentication message-digest
exit
interface tunnel.0
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 P@ssw0rd
exit
interface fw
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 P@ssw0rd
exit
write memory
```

Подожди 30–40 секунд. Соседа два, оба Full: `10.10.10.1` на `tunnel.0` и адрес Ideco на `fw`. Один сосед — OSPF на Ideco ещё не включён.

```
show ip ospf neighbor
show ip ospf interface fw
show ip route ospf
```

## 8. NAT

Пул обязан включать адрес Ideco `10.20.10.2` и сети BR-SRV / BR-CLI. Иначе таблица пустая.

```
interface isp
ip nat outside
exit
interface fw
ip nat inside
exit
ip nat pool BR 10.20.10.1-10.20.30.6
ip nat source dynamic inside-to-outside pool BR overload interface isp
write memory
```

На BR-SRV оставь `ping 77.88.8.8`. В таблице: `10.20.20.2` уходит как `172.16.2.2`.

```
show ip nat translations
```

Если пусто — переписать пул и правило. На пробнике NAT здесь ожил только после `reload`.

```
no ip nat source dynamic inside-to-outside pool BR overload interface isp
no ip nat pool BR
ip nat pool BR 10.20.10.1-10.20.30.6
ip nat source dynamic inside-to-outside pool BR overload interface isp
write memory
reload
```

## 9. Часовой пояс

```
ntp timezone utc+3
write memory
```
