# HQ-SRV

Сервер головного офиса. Адрес `10.10.100.2/27`, шлюз `10.10.100.1`. Садись сюда после HQ-RTR и HQ-SW.

## 1. Имя

```
hostnamectl set-hostname hq-srv.au-team.irpo
```

В `/etc/sysconfig/network`:

```
HOSTNAME=hq-srv.au-team.irpo
```

```
exec bash
```

## 2. Адрес, шлюз, DNS

```
echo "10.10.100.2/27" > /etc/net/ifaces/ens3/ipv4address
echo "default via 10.10.100.1" > /etc/net/ifaces/ens3/ipv4route
echo "nameserver 10.10.100.2" > /etc/net/ifaces/ens3/resolv.conf
echo "nameserver 77.88.8.8" >> /etc/net/ifaces/ens3/resolv.conf
systemctl restart network
```

Проверка: свой шлюз, затем интернет. Второй пинг потом оставь включённым, когда будешь смотреть NAT на HQ-RTR.

```
ping 10.10.100.1
ping 77.88.8.8
```

## 3. Учётка sshuser

UID `2026`, пароль `P@ssw0rd`, sudo без пароля.

```
useradd sshuser -u 2026
passwd sshuser
usermod -aG wheel sshuser
echo "sshuser ALL=(ALL:ALL) NOPASSWD: ALL" >> /etc/sudoers
```

## 4. SSH

Порт по договорённости `2027`, не 2026. Вход только `sshuser`, две попытки, баннер.

```
echo "Port 2027" >> /etc/openssh/sshd_config
echo "AllowUsers sshuser" >> /etc/openssh/sshd_config
echo "MaxAuthTries 2" >> /etc/openssh/sshd_config
echo "Banner /etc/openssh/banner" >> /etc/openssh/sshd_config
echo "Authorized access only" > /etc/openssh/banner
systemctl restart sshd
```

## 5. DNS

Ставь, когда адреса уже пингуются. Зоны не набирай с нуля, если файлы уже лежат в `/etc/bind/zone/`.

```
apt-get install -y bind bind-utils
mcedit /etc/bind/options.conf
```

В `options.conf` должны быть такие строки:

```
listen-on { 127.0.0.1; 10.10.100.2; };
forward only;
forwarders { 77.88.8.8; };
allow-query { any; };
allow-query-cache { any; };
allow-recursion { any; };
```

Файлы зон:

```
mcedit /etc/bind/zone/10.10.in-addr.arpa
mcedit /etc/bind/zone/16.172.in-addr.arpa
mcedit /etc/bind/zone/au-team.irpo.zone
systemctl enable --now bind.service
```

- `10.10.in-addr.arpa` — обратная зона `10.10.0.0/16`: hq-rtr, hq-srv, hq-cli
- `16.172.in-addr.arpa` — обратная зона `172.16.0.0/16`: hq-rtr
- `au-team.irpo.zone` — прямая зона, все A-записи и docker/web на ISP

## 6. Часовой пояс

```
timedatectl set-timezone Europe/Moscow
```
