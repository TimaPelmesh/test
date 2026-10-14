# BR-CLI

Клиент филиала. Делай только если эта машина есть на стенде. Адрес вручную, не по DHCP.

## 1. Имя

```
su -
hostnamectl set-hostname br-cli.au-team.irpo
```

В `/etc/sysconfig/network`:

```
HOSTNAME=br-cli.au-team.irpo
```

```
exec bash
```

## 2. Адрес

Параметры IPv4 → Вручную.

- адрес `10.20.30.2`
- маска `29`
- шлюз `10.20.30.1`
- DNS `10.10.100.2`, `77.88.8.8`
- домен `au-team.irpo`

Шлюз — адрес Ideco на `iif1`. Его нужно задать на BR-FW до этой машины.
