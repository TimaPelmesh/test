# BR-SRV

Сервер филиала. Адрес `10.20.20.2/28`, шлюз `10.20.20.1` на Ideco. Садись после BR-FW.

## 1. Имя

```
hostnamectl set-hostname br-srv.au-team.irpo
```

В `/etc/sysconfig/network`:

```
HOSTNAME=br-srv.au-team.irpo
```

```
exec bash
```

## 2. Адрес, шлюз, DNS

DNS — сервер головного офиса и Яндекс.

```
echo "10.20.20.2/28" > /etc/net/ifaces/ens3/ipv4address
echo "default via 10.20.20.1" > /etc/net/ifaces/ens3/ipv4route
echo "nameserver 10.10.100.2" > /etc/net/ifaces/ens3/resolv.conf
echo "nameserver 77.88.8.8" >> /etc/net/ifaces/ens3/resolv.conf
systemctl restart network
```

Проверка шлюза и интернета. Второй пинг оставь включённым, когда смотришь NAT на BR-RTR.

```
ping 10.20.20.1
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

Порт `2027`.

```
echo "Port 2027" >> /etc/openssh/sshd_config
echo "AllowUsers sshuser" >> /etc/openssh/sshd_config
echo "MaxAuthTries 2" >> /etc/openssh/sshd_config
echo "Banner /etc/openssh/banner" >> /etc/openssh/sshd_config
echo "Authorized access only" > /etc/openssh/banner
systemctl restart sshd
```

## 5. Часовой пояс

```
timedatectl set-timezone Europe/Moscow
```
