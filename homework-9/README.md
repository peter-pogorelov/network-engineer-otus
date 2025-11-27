# Часть 1. Создание сети и настройка основных параметров устройства

![img.png](images/img.png)

## 1. Создайте сеть согласно топологии.

## 2. Произведите базовую настройку маршрутизаторов.
> OK, по аналогии с прошлыми ДЗ

## 3. Настройте базовые параметры каждого коммутатора.
> OK, по аналогии с прошлыми ДЗ

# Часть 2. Настройка и проверка базовой работы протокола OSPFv2 для одной области

## 1. Настройте адреса интерфейса и базового OSPFv2 на каждом маршрутизаторе.

a. Настройте адреса интерфейсов на каждом маршрутизаторе, как показано в таблице адресации выше.

```bash
R1(config)#int g0/0/1
R1(config-if)#ip add 10.53.0.1 255.255.255.0
R1(config-if)#int lo 1
R1(config-if)#ip add 172.16.1.1 255.255.255.0
```

```bash
R2(config)#int g0/0/1
R2(config-if)#ip add 10.53.0.2 255.255.255.0
R2(config-if)#int lo 1
R2(config-if)#ip add 192.168.1.1 255.255.255.0
```

b.Перейдите в режим конфигурации маршрутизатора OSPF, используя идентификатор процесса 56.
c.Настройте статический идентификатор маршрутизатора для каждого маршрутизатора (1.1.1.1 для R1, 2.2.2.2 для R2).
d.Настройте инструкцию сети для сети между R1 и R2, поместив ее в область 0.


```bash
R1(config)#router ospf 56
R1(config-router)#router-id 1.1.1.1
R1(config-router)#int g0/0/1
R1(config-if)#ip ospf 56 area 0
```

```bash
R2(config)#router ospf 56
R2(config-router)#router-id 2.2.2.2
R2(config-router)#int g0/0/1
R2(config-if)#ip ospf 56 area 0
```

e. Только на R2 добавьте конфигурацию, необходимую для объявления сети Loopback 1 в область OSPF 0.

```bash
R2(config-if)#int lo 1
R2(config-if)#ip ospf 56 area 0
```

f. Убедитесь, что OSPFv2 работает между маршрутизаторами. Выполните команду, чтобы убедиться, что R1 и R2 сформировали смежность.

> Проверяю соседей каждого маршрутизатора
```bash
R1#show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/BDR        00:00:30    10.53.0.2       GigabitEthernet0/0/1
```

```bash
R2#show ip ospf nei
Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           1   FULL/DR         00:00:35    10.53.0.1       GigabitEthernet0/0/1
```

**Вопрос:**
Какой маршрутизатор является DR? Какой маршрутизатор является BDR? Каковы критерии отбора?
> Роутер R1(id=1.1.1.1) является DR, а роутер R2(id=2.2.2.2) является BDR. 
> Ситуация странная, так как в роли DR должен по идее быть выбран роутер с наибольшим RID. Но что-то пошло не так.

g. На R1 выполните команду show ip route ospf, чтобы убедиться, что сеть R2 Loopback1 присутствует в таблице маршрутизации

```bash
R1# show ip route ospf
     192.168.1.0/32 is subnetted, 1 subnets
O       192.168.1.1 [110/2] via 10.53.0.2, 00:00:21, GigabitEthernet0/0/1
```

h. Запустите Ping до  адреса интерфейса R2 Loopback 1 из R1. Выполнение команды ping должно быть успешным.

```bash
R1#ping 192.168.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
```

# Часть 3. Оптимизация и проверка конфигурации OSPFv2 для одной области
## 1. Реализация различных оптимизаций на каждом маршрутизаторе.

a. На R1 настройте приоритет OSPF интерфейса G0/0/1 на 50, чтобы убедиться, что R1 является назначенным маршрутизатором.
```bash
R1(config)#int g0/0/1
R1(config-if)#ip ospf priority 50
```

b. Настройте таймеры OSPF на G0/0/1 каждого маршрутизатора для таймера приветствия, составляющего 30 секунд.
```bash
R1(config-if)#int g0/0/1
R1(config-if)#ip ospf hello-interval 30
```

```bash
R2(config-if)#int g0/0/1
R2(config-if)#ip ospf hello-interval 30
```

с. На R1 настройте статический маршрут по умолчанию, который использует интерфейс Loopback 1 в качестве интерфейса выхода. Затем распространите маршрут по умолчанию в OSPF. Обратите внимание на сообщение консоли после установки маршрута по умолчанию.
```bash
R1(config)#ip route 0.0.0.0 0.0.0.0 lo 1
%Default route without gateway, if not a point-to-point interface, may impact performance
R1(config)#router ospf 56
R1(config-router)#default-information originate
```

d. Добавьте конфигурацию, необходимую для OSPF для обработки R2 Loopback 1 как сети точка-точка. Это приводит к тому, что OSPF объявляет Loopback 1 использует маску подсети интерфейса.
> Вот тут криво как-то перевели, мне пришлось гуглить лабу чтобы понять что тут вообще имеется в виду :(
> 
> On R2 only, add the configuration necessary for OSPF to treat R2 Loopback 1 like a point-to-point network. This results in OSPF advertising Loopback 1 using the interface subnet mask.

```bash
R2(config)#int lo 1
R2(config-if)#ip ospf network point-to-point
```

e. Только на R2 добавьте конфигурацию, необходимую для предотвращения отправки объявлений OSPF в сеть Loopback 1.
```bash
R2(config)#router ospf 56
R2(config-router)#passive-interface lo 1
```

f. Измените базовую пропускную способность для маршрутизаторов. После этой настройки перезапустите OSPF с помощью команды clear ip ospf process . Обратите внимание на сообщение консоли после установки новой опорной полосы пропускания.

```bash
R1(config)#router ospf 56
R1(config-router)#auto-cost reference-bandwidth 1000
% OSPF: Reference bandwidth is changed.
        Please ensure reference bandwidth is consistent across all routers.
R1(config-router)#do clear ip ospf process
Reset ALL OSPF processes? [no]: yes
03:21:47: %OSPF-5-ADJCHG: Process 56, Nbr 2.2.2.2 on GigabitEthernet0/0/1 from FULL to DOWN, Neighbor Down: Adjacency forced to reset
03:21:47: %OSPF-5-ADJCHG: Process 56, Nbr 2.2.2.2 on GigabitEthernet0/0/1 from FULL to DOWN, Neighbor Down: Interface down or detached
03:22:03: %OSPF-5-ADJCHG: Process 56, Nbr 2.2.2.2 on GigabitEthernet0/0/1 from LOADING to FULL, Loading Done
```

```bash
R2(config)#router ospf 56
R2(config-router)#auto-cost reference-bandwidth 1000
% OSPF: Reference bandwidth is changed.
        Please ensure reference bandwidth is consistent across all routers.
R2(config-router)#do clear ip ospf process
Reset ALL OSPF processes? [no]: yes
03:21:37: %OSPF-5-ADJCHG: Process 56, Nbr 1.1.1.1 on GigabitEthernet0/0/1 from FULL to DOWN, Neighbor Down: Adjacency forced to reset
03:21:37: %OSPF-5-ADJCHG: Process 56, Nbr 1.1.1.1 on GigabitEthernet0/0/1 from FULL to DOWN, Neighbor Down: Interface down or detached
03:21:40: %OSPF-5-ADJCHG: Process 56, Nbr 1.1.1.1 on GigabitEthernet0/0/1 from LOADING to FULL, Loading Done
```

## 2. Убедитесь, что оптимизация OSPFv2 реализовалась.

a. Выполните команду show ip ospf interface g0/0/1 на R1 и убедитесь, что приоритет интерфейса установлен равным 50, а временные интервалы — Hello 30, Dead 120, а тип сети по умолчанию — Broadcast
```bash
R1#show ip ospf interface g0/0/1

GigabitEthernet0/0/1 is up, line protocol is up
  Internet address is 10.53.0.1/24, Area 0
  Process ID 56, Router ID 1.1.1.1, Network Type BROADCAST, Cost: 10.   << -- Тип сети = Broadcast
  Transmit Delay is 1 sec, State DR, Priority 50                        << -- Приоритет = 50
  Designated Router (ID) 1.1.1.1, Interface address 10.53.0.1
  Backup Designated Router (ID) 2.2.2.2, Interface address 10.53.0.2
  Timer intervals configured, Hello 30, Dead 40, Wait 40, Retransmit 5. << -- Hello = 30, про DEAD не было речи в задании :)
    Hello due in 00:00:14
  Index 1/1, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 1, Adjacent neighbor count is 1
    Adjacent with neighbor 2.2.2.2  (Backup Designated Router)
  Suppress hello for 0 neighbor(s)
```

b. На R1 выполните команду show ip route ospf, чтобы убедиться, что сеть R2 Loopback1 присутствует в таблице маршрутизации.
```bash
R1#show ip route ospf
O    192.168.1.0 [110/10] via 10.53.0.2, 00:07:44, GigabitEthernet0/0/1
```

c. Введите команду show ip route ospf на маршрутизаторе R2. Единственная информация о маршруте OSPF должна быть распространяемый по умолчанию маршрут R1.
```bash
R2#show ip route ospf
O*E2 0.0.0.0/0 [110/1] via 10.53.0.1, 00:04:34, GigabitEthernet0/0/1
```

d. Запустите Ping до адреса интерфейса R1 Loopback 1 из R2. Выполнение команды ping должно быть успешным.
```bash
R2>ping 172.16.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.16.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/5/16 ms
```

Почему стоимость OSPF для маршрута по умолчанию отличается от стоимости OSPF в R1 для сети 192.168.1.0/24?

> Стоимость до 192.168.1.0 = стоимости канала между роутерами. Стоимость до R1 Lo1 = 1 связана с тем что это внешний маршрут второго типа, на лекции упоминалось что у них не растет метрика при переходе от маршрутизатора к маршрутизатору.