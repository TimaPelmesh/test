# HQ-SW

Коммутатор головного офиса. Если мост уже есть в заготовке стенда — не создавай заново, только сверь вывод.

`ens3` — транк на HQ-RTR, VLAN 100, 200, 999. `ens4` — доступ к HQ-SRV, VLAN 100. `ens5` — доступ к HQ-CLI, VLAN 200. `MGMT` — внутренний порт VLAN 999.

## 1. Имя

Только если это отдельная виртуальная машина со своей ОС.

```
hostnamectl set-hostname hq-sw
```

В `/etc/sysconfig/network`:

```
HOSTNAME=hq-sw
```

```
exec bash
```

## 2. Мост

Теги VLAN на стороне HQ-RTR уже заданы в `service-instance` порта `ge1`. Здесь только порты моста.

```
ovs-vsctl add-br hq-sw
ovs-vsctl add-port hq-sw ens3 trunks=100,200,999
ovs-vsctl add-port hq-sw ens4 tag=100
ovs-vsctl add-port hq-sw ens5 tag=200
ovs-vsctl add-port hq-sw MGMT tag=999 -- set interface MGMT type=internal
ovs-vsctl show
```

Вывод `ovs-vsctl show` идёт в отчёт.

Часовой пояс на виртуальном коммутаторе без своей ОС не ставится.
