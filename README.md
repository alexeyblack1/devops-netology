# devops-netology
Домашнее задание по лекции "Работа в терминале (лекция 2)
1. Какого типа команда cd? Попробуйте объяснить, почему она именно такого типа; опишите ход своих мыслей, если считаете что она могла бы быть другого типа.

type -t cd
builtin

Команда cd - встроенная. Могла бы быть и файлом, как , например "cp", но отдельного файла нет:
vagrant@vagrant:~$ type -a cd
cd is a shell builtin 

2. Какая альтернатива без pipe команде grep <some_string> <some_file> | wc -l? 

vagrant@vagrant:~$ grep History .viminfo | wc -l
6
Альтернатива: 
vagrant@vagrant:~$ grep -c History .viminfo
6

3. Какой процесс с PID 1 является родителем для всех процессов в вашей виртуальной машине Ubuntu 20.04?

vagrant@vagrant:~$ pstree -g -A
systemd(1)-+-VBoxService(786)-+-{VBoxService}(786)
           |                  |-{VBoxService}(786)
           |                  |-{VBoxService}(786)
           |                  |-{VBoxService}(786)
           |                  |-{VBoxService}(786)
           |                  |-{VBoxService}(786)
           |                  |-{VBoxService}(786)
           |                  `-{VBoxService}(786)
           |-accounts-daemon(600)-+-{accounts-daemon}(600)
           |                      `-{accounts-daemon}(600)
           |-agetty(949)
           |-agetty(961)
           |-atd(675)
Процесс systemd - родитель и имеет PID 1

4. Как будет выглядеть команда, которая перенаправит вывод stderr ls на другую сессию терминала?

vagrant@vagrant:~$ ls 2>/dev/pts/1

5. Получится ли одновременно передать команде файл на stdin и вывести ее stdout в другой файл? Приведите работающий пример.
Да. Пример передачи команде cat на вход file1 и запись stdout в file2

vagrant@vagrant:~$ cat file01
001
vagrant@vagrant:~$ cat file02

vagrant@vagrant:~$ cat < file01 > file02
vagrant@vagrant:~$ cat file02
001

6.Получится ли вывести находясь в графическом режиме данные из PTY в какой-либо из эмуляторов TTY? 
Сможете ли вы наблюдать выводимые данные?
Получится, т.к. псевдотерминал состоит из следующих двух виртуальных TTY устройств:
1) PTY master (PTM) — ведущая часть псевдотерминала. 
2) PTY slave (PTS) — ведомая часть псевдотерминала. Используется программой, запущенной внутри терминала, для чтения ввода с клавиатуры и отображения вывода на экран. 
PTS также является терминальным устройством (TTY устройством), но разница между PTS устройством и действительным TTY устройством в том, что PTS устройство ввод получает не с клавиатуры, а с master устройства, а вывод идет не на дисплей, а на master устройство. PTS устройство находится по пути /dev/pts/N

vagrant@vagrant:~$ tty
/dev/pts/0
vagrant@vagrant:~$ echo Hello > /dev/pts/1
Hello отобразится в терминале 1 (если он запущен)

7.Выполните команду bash 5>&1. К чему она приведет? Что будет, если вы выполните echo netology > /proc/$$/fd/5? Почему так происходит?

Команда bash 5>&1 запускает bash, создает  дексриптор 5 и перенаправляет его в stdout. 
Команда echo 'netology' > /proc/$$/fd/5 выведет на экран  netology. Т.к. в bash 5-й дескриптор, в который выводится результат  echo 'netology' , перенаправлен в stdout. $$ - это  обращение к PID нашей shell. 
В fd - записи на каждый файл, который открыт процессом. Имя записи - ссылка на реальный файл по номеру дескриптора (номер 5). 

8. Получится ли в качестве входного потока для pipe использовать только stderr команды, не потеряв при этом отображение stdout на pty? Напоминаем: по умолчанию через pipe передается только stdout команды слева от | на stdin команды справа. Это можно сделать, поменяв стандартные потоки местами через промежуточный новый дескриптор, который вы научились создавать в предыдущем вопросе.

Получится. Меняем  местами stdout и stderr. Команда bash 3>&1 1>&2 2>&3 запустит bash в котором дескрипторы stdout и stderr будут поменяны местами через промежуточный дескриптор 3 и  в stdin через pipe будет передаваться уже stderr. 
Пример команды ls без вывода в консоль,  с выводом stderr в файл через pipe:
vagrant@vagrant:~$ ls -J 3>&1 1>&2 2>&3 | tee -p file1 > /dev/null
vagrant@vagrant:~$ cat file1
ls: invalid option -- 'J'
Try 'ls --help' for more information.

9. Что выведет команда cat /proc/$$/environ? Как еще можно получить аналогичный по содержанию вывод?
Команда выведет переменные оболочки. Переменная $$ хранит идентификатор процесса оболочки. Аналогичный вывод можно командой env или set.

10. Используя man, опишите что доступно по адресам /proc/<PID>/cmdline, /proc/<PID>/exe.
cmdline - Этот файл содержит полную командную строку запуска процесса, кроме тех процессов, что полностью ушли в своппинг, а также тех, что превратились в зомби. В этих двух случаях в файле ничего нет, то есть чтение этого файла вернет 0 символов. Аргументы командной строки в этом файле указаны как список строк, каждая из которых завешрается нулевым символом, с добавочным нулевым байтом после последней строки.
exe -  является символьной ссылкой, содержащей фактическое полное имя выполняемого файла. Символьная ссылка exe может использоваться обычным образом - при попытке открыть exe будет открыт исполняемый файл. Можно  ввести /proc/[number]/exe чтобы запустить другую копию процесса такого же как и процесс с номером <PID>

11.Узнайте, какую наиболее старшую версию набора инструкций SSE поддерживает ваш процессор с помощью /proc/cpuinfo.
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht s
yscall nx rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc cpuid tsc_known_freq pni pclmulqdq ssse3 cx16 pcid
sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx rdrand hypervisor lahf_lm abm invpcid_single pti fsgsbase avx2 invpcid

Старшая версия sse4_2

12. При открытии нового окна терминала и vagrant ssh создается новая сессия и выделяется pty. Это можно подтвердить командой tty, которая упоминалась в лекции 3.2. Однако:

vagrant@netology1:~$ ssh localhost 'tty'
not a tty
Почитайте, почему так происходит, и как изменить поведение.

При создании новой сессии по ssh не выделяется псевдотерминал, для  нового псевдотерминала 
-t      Force pseudo-terminal allocation.  This can be used to execute arbitrary screen-based programs
 on a remote machine, which can be very useful, e.g. when implementing menu services.  Multiple -t options force
        tty allocation, even if ssh has no local tty.
Принудительное выделение псевдотерминала. Это можно использовать для выполнения произвольных экранных программ на удаленной машине, что может быть очень полезно, например при реализации услуг меню. Множественные параметр -t принудительное выделение tty, даже если ssh не имеет локального tty.

vagrant@netology1:~$ ssh -t localhost 'tty' 

13. Бывает, что есть необходимость переместить запущенный процесс из одной сессии в другую. Попробуйте сделать это, воспользовавшись reptyr. Например, так можно перенести в screen процесс, который вы запустили по ошибке в обычной SSH-сессии.
Последовательность из терминала:
[1]+  Stopped                 top
vagrant@vagrant:~$ disown -r
vagrant@vagrant:~$ jobs
[1]   Stopped                 top
vagrant@vagrant:~$ screen -list
There is a screen on:
        1733.pts-0.vagrant      (06/09/2021 04:53:34 PM)        (Detached)
1 Socket in /run/screen/S-vagrant.
vagrant@vagrant:~$ jobs -l
[1]   1772 Stopped (signal)        top
vagrant@vagrant:~$ reptyr 1772
Unable to attach to pid 1772: Operation not permitted
The kernel denied permission while attaching. If your uid matches
the target's, check the value of /proc/sys/kernel/yama/ptrace_scope.
For more information, see /etc/sysctl.d/10-ptrace.conf

[1]+  Stopped                 top
vagrant@vagrant:~$ reptyr -L 1772
Opened a new pty: /dev/pts/2

14. sudo echo string > /root/new_file не даст выполнить перенаправление под обычным пользователем, так как перенаправлением занимается процесс shell'а, который запущен без sudo под вашим пользователем. Для решения данной проблемы можно использовать конструкцию echo string | sudo tee /root/new_file. Узнайте что делает команда tee и почему в отличие от sudo echo команда с sudo tee будет работать.

tee - read from standard input and write to standard output and files
 Команда tee читает stdin пишет в stdout и в файл.
Выполня  ее от sudo, позволит перезаписать файл.
