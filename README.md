                                      Домашнее задание к занятию "3.4. Операционные системы, лекция 2"
1.**На лекции мы познакомились с node_exporter. В демонстрации его исполняемый файл запускался в background. Этого достаточно для демо, но не для настоящей production-системы, где процессы должны находиться под внешним управлением. Используя знания из лекции по systemd, создайте самостоятельно простой unit-файл для node_exporter:
поместите его в автозагрузку,
предусмотрите возможность добавления опций к запускаемому процессу через внешний файл (посмотрите, например, на systemctl cat cron),
удостоверьтесь, что с помощью systemctl процесс корректно стартует, завершается, а после перезагрузки автоматически поднимается.**
vagrant@vagrant:$ cd /tmp
vagrant@vagrant:/tmp$ curl -LO https://github.com/prometheus/node_exporter/releases/download/v1.1.2/node_exporter-1.1.2.linux-amd64.tar.gz
vagrant@vagrant:/tmp$ sudo nano /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter
After=network.target
[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter
[Install]
WantedBy=multi-user.target
После добавления в автозагрузку и проверки статуса - вот так:
vagrant@vagrant:$ sudo systemctl status node_exporter
? node_exporter.service - Node Exporter
Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; vendor preset: enabled)
Active: failed (Result: exit-code) since Tue 2021-06-22 18:21:00 UTC; 9min ago
Process: 658 ExecStart=/usr/local/bin/node_exporter (code=exited, status=217/USER)
Main PID: 658 (code=exited, status=217/USER)
Jun 22 18:21:00 vagrant systemd[1]: Started Node Exporter.
Jun 22 18:21:00 vagrant systemd[658]: node_exporter.service: Failed to determine user credentials: No such process
Jun 22 18:21:00 vagrant systemd[658]: node_exporter.service: Failed at step USER spawning /usr/local/bin/node_exporter:
Jun 22 18:21:00 vagrant systemd[1]: node_exporter.service: Main process exited, code=exited, status=217/USER
Jun 22 18:21:00 vagrant systemd[1]: node_exporter.service: Failed with result 'exit-code'.

**Ознакомьтесь с опциями node_exporter и выводом /metrics по-умолчанию. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.**

Не получается выполнить первый пункт задания.

**Установите в свою виртуальную машину Netdata. Воспользуйтесь готовыми пакетами для установки (sudo apt install -y netdata). После успешной установки:
в конфигурационном файле /etc/netdata/netdata.conf в секции [web] замените значение с localhost на bind to = 0.0.0.0,
добавьте в Vagrantfile проброс порта Netdata на свой локальный компьютер и сделайте vagrant reload:
config.vm.network "forwarded_port", guest: 19999, host: 19999
После успешной перезагрузки в браузере на своем ПК (не в виртуальной машине) вы должны суметь зайти на localhost:19999. Ознакомьтесь с метриками, которые по умолчанию собираются Netdata и с комментариями, которые даны к этим метрикам.

Отражаются следующие метрики:
cpu
load
disk
ram
swap
network
processes
idlejitter
interrupts
softirqs
softnet
entropy
uptime
ipc semaphores
ipc shared memory

**Можно ли по выводу dmesg понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?**

Можно, достаточно посмотреть:
vagrant@vagrant:$ dmesg | grep -i virtual
[    0.000000] DMI: innotek GmbH VirtualBox/VirtualBox, BIOS VirtualBox 12/01/2006
[    0.002846] CPU MTRRs all blank - virtualized system.
[    0.118588] Booting paravirtualized kernel on KVM
[    3.768753] systemd[1]: Detected virtualization oracle.


**Как настроен sysctl fs.nr_open на системе по-умолчанию? Узнайте, что означает этот параметр. Какой другой существующий лимит не позволит достичь такого числа (ulimit --help)?**
vagrant@vagrant:$ sysctl fs.nr_open
fs.nr_open = 1048576
Это максимальный лимит открытых файловых дескрипторов.
Его не позволит достичь значение, указанное в ulimit:
-n        the maximum number of open file descriptors
vagrant@vagrant:$ ulimit -n
1024


**Запустите любой долгоживущий процесс (не ls, который отработает мгновенно, а, например, sleep 1h) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под PID 1 через nsenter. Для простоты работайте в данном задании под root (sudo -i). Под обычным пользователем требуются дополнительные опции (--map-root-user) и т.д.**


Запускаем процесс top
unshare -f --pid --mount-proc top
Из другого терминала пробуем посмотреть:
root@vagrant:# lsns
vagrant@vagrant:$ sudo -i
root@vagrant:# nsenter -t $(pidof top) -m -p ps ax
PID TTY      STAT   TIME COMMAND
1 pts/0    S+     0:00 top
2 pts/1    R+     0:00 ps ax

**Найдите информацию о том, что такое :(){ :|:& };:. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04 (это важно, поведение в других ОС не проверялось). Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов dmesg расскажет, какой механизм помог автоматической стабилизации. Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?**

:(){ :|:& };: - Это пример  так называемой fork-бомбы. Она  порождает большое количество собственных копий и тем самым пытается заполнить свободное место в списке активных процессов операционной системы. После заполнения списка процессов становится невозможным старт полезной программы. Даже если какой-либо другой процесс прекратит работу и место в списке процессов освободится, то старт полезной программы маловероятен, так как множество других копий fork-бомбы уже ждут возможности запустить свою очередную копию.
Кроме заполнения списка процессов, возможны также стратегии заполнения виртуальной памяти, процессорного времени, сокетов и других системных ресурсов. Результатом исчерпания этих ресурсов становится замедление работы или практически остановка операционной системы и/или полезных программ (зависание компьютера).
: () {
: |: &
};:
:() - Определяет  функцию под названием :. Эта функция не принимает аргументов.
:|: - Затем вызывает себя, используя метод, называемый рекурсией, и направит вывод на другой вызов функции ‘:’. Хуже всего то, что функция вызывается дважды, чтобы "взорвать"  систему.
& - переводит вызов функции в фоновый режим, чтобы дочерний процесс вообще не мог умереть и начал есть системные ресурсы.
; - Завершит определение функции.
: - Вызывает (запускает) функцию снова.
Вот к чему это привело:
-bash: fork: retry: Resource temporarily unavailable
-bash: fork: Resource temporarily unavailable
-bash: fork: Resource temporarily unavailable
Чтобы изменить число процессов, которое можно создать в сессии до 5000 процессов, используем следующую команду
ulimit -S -u 5000
max user processes              (-u) 5000
[  398.502697] cgroup: fork rejected by pids controller in /user.slice/user-1000.slice/session-4.scope
