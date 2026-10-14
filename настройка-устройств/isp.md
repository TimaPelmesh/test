# ISP

Linux-маршрутизатор между HQ-RTR, BR-RTR и интернетом. Делай первым.

Адреса на стенде обычно уже стоят: к HQ-RTR `172.16.1.1/28`, к BR-RTR `172.16.2.1/28`, аплинк по DHCP.

## 1. Имя

```
hostnamectl set-hostname isp
```

В `/etc/sysconfig/network` строка:

```
HOSTNAME=isp
```

```
exec bash
```

## 2. Пересылка пакетов

Без `ip_forward` GRE между роутерами через ISP не пойдёт, OSPF тоже.

```
sed -i "s/net.ipv4.ip_forward = 0/net.ipv4.ip_forward = 1/" /etc/net/sysctl.conf
sysctl -w net.ipv4.ip_forward=1
```

## 3. Маскарад в интернет

Имя аплинка (интерфейс с адресом по DHCP) смотри в выводе и подставь вместо `ens3`.

```
ip -c -br a
iptables -t nat -A POSTROUTING -s 172.16.1.0/28 -o ens3 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 172.16.2.0/28 -o ens3 -j MASQUERADE
iptables-save > /etc/sysconfig/iptables
systemctl enable --now iptables.service
```

## 4. Проверка

Должно быть `1` и два правила MASQUERADE.

```
sysctl net.ipv4.ip_forward
iptables -t nat -L POSTROUTING -n -v
```

## 5. Часовой пояс

```
timedatectl set-timezone Europe/Moscow
```
