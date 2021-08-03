                               Домашнее задание к занятию "3.6. Компьютерные сети, лекция 1"
**1.Необязательное задание: можно посмотреть целый фильм в консоли telnet towel.blinkenlights.nl :)**

**2.Узнайте о том, сколько действительно независимых (не пересекающихся) каналов есть в разделяемой среде WiFi при работе на 2.4 ГГц. Стандарты с полосой 5 ГГц более актуальны, но регламенты на 5 ГГц существенно различаются в разных странах, а так же не раз обновлялись. В качестве дополнительного вопроса вне зачета, попробуйте найти актуальный ответ и на этот вопрос.**

Не перекающимися каналами в диапазоне 2.4 ГГц являются каналы 1-й, 6-й и 11-й. в России можно считать не перекающимися и каналы 2-7-12 и 3-8-1, но в большинстве стран в этом диапазон доступны только 
   11 каналов, поэтому действительно независимыми можно считать 1-6-11. 
   
   5 ГГц - это не единый диапазон частот, а ряд диапазонов. Выделяемые для нелицензируемой работы полосы частот называются UNII (Нелицензируемая национальная информационная инфраструктура).
      Диапазон содержит 25 непересекающихся каналов.  В России на текущий момент времени из диапазона UNII-2 Extended разрешено использование только четырех каналов 132-144, таким образом у нас доступно всего 17 непересекающихся каналов, по четыре в каждой из полос UNII и один в ISM.
  

**3. Адрес канального уровня – MAC адрес – это 6 байт, первые 3 из которых называются OUI – Organizationally Unique Identifier или уникальный юидентификатор организации. Какому производителю принадлежит MAC 38:f9:d3:55:55:79?**

Apple, Inc

**4.Каким будет payload TCP сегмента, если Ethernet MTU задан в 9001 байт, размер заголовков IPv4 – 20 байт, а TCP – 32 байта?**

Максимальный размер payload TCP (MSS) = MTU -(минус) размер заголовка IP -(минус) размер заголовка TCP, соответственно в данном случае MSS будет равен 
9001-20-32=8949

**5. Может ли во флагах TCP одновременно быть установлены флаги SYN и FIN при штатном режиме работы сети? Почему да или нет?**

 Комбинация флага SYN и FIN, устанавливаемого в заголовке TCP, является недопустимой при штатной работе и относится к категории комбинации недопустимого / ненормального флага, поскольку она требует как установления соединения (через SYN), так и прекращения соединения (через FIN).

**6. ss -ula sport = :53 на хосте имеет следующий вывод:**
```
State           Recv-Q          Send-Q                   Local Address:Port                     Peer Address:Port          Process
UNCONN          0               0                        127.0.0.53%lo:domain                        0.0.0.0:*
```
**Почему в State присутствует только UNCONN, и может ли там присутствовать, например, TIME-WAIT?**

Состояние UNCONN - это значит отключен, так как этот порт ничем не используется.  В данном выводе TIME-WAIT присутствовать не может, так как мы используем ключ -u, а это значит,
   выводится информация для udp-сокетов, а  состояние TIME-WAIT может быть только у tcp-сокета.
  В данном выводе TIME-WAIT присутствовать не может, так как мы используем ключ -u, а это значит,
 что  выводится информация для udp-сокетов, а  состояние TIME-WAIT может быть только у tcp-сокета. 
Если не исключать UDP результат будет:
```
vagrant@vagrant:~$ ss -la sport = :53
Netid      State       Recv-Q      Send-Q           Local Address:Port              Peer Address:Port      Process

udp        UNCONN      0           0                127.0.0.53%lo:domain                 0.0.0.0:*

tcp        LISTEN      0           4096             127.0.0.53%lo:domain                 0.0.0.0:*
```
**7.Обладая знаниями о том, как штатным образом завершается соединение (FIN от инициатора, FIN-ACK от ответчика, ACK от инициатора), опишите в каких состояниях будет находиться TCP соединение в каждый момент времени на клиенте и на сервере при завершении. Схема переходов состояния соединения вам в этом поможет.**

Начальное состояние|Действие|Переход в состояние|Начальное состояние|Действие|Переход в состояние
-------------------|--------|-------------------|-------------------|--------|-------------------
ESTABLISHED|FIN-WAIT-1|ESTABLISHED| |
FIN-WAIT-1| |ESTABLISHED|CLOSE-WAIT
FIN-WAIT-1|FIN-WAIT-2|CLOSE-WAIT| |
FIN-WAIT-2| |CLOSE-WAIT|LAST-ACK
FIN-WAIT-2| |LAST-ACK| |TIME-WAIT| |LAST-ACK|CLOSED
TIME-WAIT|CLOSED|CLOSED| |
CLOSED| |CLOSED| |


**8.TCP порт – 16 битное число. Предположим, 2 находящихся в одной сети хоста устанавливают между собой соединения. Каким будет теоретическое максимальное число соединений, ограниченное только лишь параметрами L4, которое параллельно может установить клиент с одного IP адреса к серверу с одним IP адресом? Сколько соединений сможет обслужить сервер от одного клиента? А если клиентов больше одного?**

Так как TCP-порт шестнадцатибитное число - максимальное количество портов составляет *65535* - это и будет максимальное количество соединений.
   
   Количество портов на сервере будет регулироваться значением 'net.ipv4.ip_local_port_range = 32768    60999'. Данный параметр указывает диапазон используемых портов клиентскими приложениями,  т. е. начало и конец диапазона. Диапазон ограничивается наличием памяти на хосте, чем больший объем памяти имеется у хоста, тем больший диапазон портов можно задать

**9.Может ли сложиться ситуация, при которой большое число соединений TCP на хосте находятся в состоянии TIME-WAIT? Если да, то является ли она хорошей или плохой? Подкрепите свой ответ пояснением той или иной оценки.**

Ситуация, когда большое число соединений TCP на хосте находятся в состоянии `TIME-WAIT` может сложиться, если соединения клиентом устанавливаются быстрее, чем порты выходят из карантина, то есть остаются в состоянии TIME_WAIT.
   Очередь TIME_WAIT будет расти, забирая системные ресурсы.

**10.Чем особенно плоха фрагментация UDP относительно фрагментации TCP?**

Протокол UDP в отличии от протокола TCP не имеет механизма подтверждения получения пакетов и механизма переупорядочивания, то есть если потеряется дефрагментированный пакет или пакеты придут не в том порядке, то исходные данные будут потеряны. 

**11.Если бы вы строили систему удаленного сбора логов, то есть систему, в которой несколько хостов отправяют на центральный узел генерируемые приложениями логи (предположим, что логи – текстовая информация), какой протокол транспортного уровня вы выбрали бы и почему? Проверьте ваше предположение самостоятельно, узнав о стандартном протоколе syslog.**

Я бы использовал протокол TCP, так как он гарантирует доставку пакетов.
   *syslog* может работать и по UDP и по TCP.

**12.Сколько портов TCP находится в состоянии прослушивания на вашей виртуальной машине с Ubuntu, и каким процессам они принадлежат?**
```
vagrant@vagrant:~$ sudo  ss -tlap
State  Recv-Q Send-Q Local Address:Port    Peer Address:Port  Process

LISTEN 0      4096         0.0.0.0:sunrpc       0.0.0.0:*      users:(("rpcbind",pid=557,fd=4),("systemd",pid=1,fd=35))

LISTEN 0      4096   127.0.0.53%lo:domain       0.0.0.0:*      users:(("systemd-resolve",pid=558,fd=13))

LISTEN 0      128          0.0.0.0:ssh          0.0.0.0:*      users:(("sshd",pid=829,fd=3))

ESTAB  0      0          10.0.2.15:ssh         10.0.2.2:57067  users:(("sshd",pid=1133,fd=4),("sshd",pid=1096,fd=4))

LISTEN 0      4096               *:9100               *:*      users:(("node_exporter",pid=627,fd=3))

LISTEN 0      4096            [::]:sunrpc          [::]:*      users:(("rpcbind",pid=557,fd=6),("systemd",pid=1,fd=37))

LISTEN 0      128             [::]:ssh             [::]:*      users:(("sshd",pid=829,fd=4))

```
В режиме прослушивания находится 6 портов. Они принадлежат процессам systemd, sshd,  node_exporter, rpcbind .

**13.Какой ключ нужно добавить в tcpdump, чтобы он начал выводить не только заголовки, но и содержимое фреймов в текстовом виде? А в текстовом и шестнадцатиричном?**

С ключом -A `tcpdump` будет выводить данные в текстовом виде. 

```
vagrant@vagrant:~/devops-netology$ sudo tcpdump -A -c 1
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
15:36:33.337690 IP vagrant.ssh > _gateway.62009: Flags [P.], seq 1364712446:1364712482, ack 19093131, win 62780, length
36
E..L.^@.@.j-
...
......9QW...#V.P..<.O..z.t.H.^.c...u<....OD8...s.L...E....%
1 packet captured
11 packets received by filter
0 packets dropped by kernel
```


**14.Попробуйте собрать дамп трафика с помощью tcpdump на основном интерфейсе вашей виртуальной машины и посмотреть его через tshark или Wireshark (можно ограничить число пакетов -c 100). Встретились ли вам какие-то установленные флаги Internet Protocol (не флаги TCP, а флаги IP)? Узнайте, какие флаги бывают. Как на самом деле называется стандарт Ethernet, фреймы которого попали в ваш дамп? Можно ли где-то в дампе увидеть OUI?**

Ограничим число пакетов 50-ю.
```
vagrant@vagrant:~/devops-netology$ sudo tcpdump -i eth0 -c 50 -w tcpdump.pcap
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
50 packets captured
50 packets received by filter
0 packets dropped by kernel

sudo tshark -r tcpdump.pcap
```
```
vagrant@vagrant:~$ sudo tcpdump -r tcpdump.pcap
reading from file tcpdump.pcap, link-type EN10MB (Ethernet)
12:14:03.218332 IP vagrant.ssh > _gateway.63722: Flags [P.], seq 3547826712:3547826748, ack 5000403, win 62780, length 3
6
12:14:03.218393 IP vagrant.ssh > _gateway.63722: Flags [P.], seq 36:112, ack 1, win 62780, length 76
12:14:03.218449 IP vagrant.ssh > _gateway.63722: Flags [P.], seq 112:148, ack 1, win 62780, length 36
12:14:03.218586 IP _gateway.63722 > vagrant.ssh: Flags [.], ack 36, win 65535, length 0
12:14:03.218586 IP _gateway.63722 > vagrant.ssh: Flags [.], ack 112, win 65535, length 0
12:14:03.218586 IP _gateway.63722 > vagrant.ssh: Flags [.], ack 148, win 65535, length 0
12:14:22.448781 IP _gateway.52118 > vagrant.ssh: Flags [S], seq 22080001, win 65535, options [mss 1460], length 0
12:14:22.448808 IP vagrant.ssh > _gateway.52118: Flags [S.], seq 822955223, ack 22080002, win 64240, options [mss 1460],
 length 0
 ```
