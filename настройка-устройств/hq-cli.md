# HQ-CLI

Клиент головного офиса. Адрес сам не задаётся: его выдаёт DHCP на HQ-RTR, пункт с пулом VLAN200. Сначала подними DHCP на роутере, потом проверяй здесь.

## 1. Имя

Терминал: ПКМ по рабочему столу → Открыть в Терминале, затем `su -`.

```
su -
hostnamectl set-hostname hq-cli.au-team.irpo
```

В `/etc/sysconfig/network`:

```
HOSTNAME=hq-cli.au-team.irpo
```

```
exec bash
```

## 2. Проверка адреса

Ожидается адрес из `10.10.200.2`–`10.10.200.14`, шлюз `10.10.200.1`, DNS `10.10.100.2`, домен `au-team.irpo`.

```
ip -c a
cat /etc/resolv.conf
```

## 3. Часовой пояс

```
timedatectl set-timezone Europe/Moscow
```
