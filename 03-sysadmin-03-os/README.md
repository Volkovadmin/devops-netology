##                        ДЗ к занятию "3.3. Операционные системы (лекция 1)" <br/> <br/>


**1.** Какой системный вызов делает команда cd? <br/> 
В прошлом ДЗ мы выяснили, что cd не является самостоятельной программой, это shell builtin, <br/> 
поэтому запустить strace непосредственно на cd не получится. <br/> 
Тем не менее, вы можете запустить strace на /bin/bash -c 'cd /tmp'. <br/> 
В этом случае вы увидите полный список системных вызовов, которые делает сам bash при старте. <br/> 
Вам нужно найти тот единственный, который относится именно к cd. <br/> 
Обратите внимание, что strace выдаёт результат своей работы в поток stderr, а не в stdout.


`chdir("/tmp")` <br/> <br/> 


**2.** Попробуйте использовать команду `file` на объекты разных типов на файловой системе. <br/> 
Например: <br/>
```shell
vagrant@netology1:~$ file /dev/tty
/dev/tty: character special (5/0)
vagrant@netology1:~$ file /dev/sda
/dev/sda: block special (8/0)
vagrant@netology1:~$ file /bin/bash
/bin/bash: ELF 64-bit LSB shared object, x86-64
```
Используя `strace` выясните, где находится база данных file на основании которой она делает свои догадки.


**Ищет пользовательские файлы и файлы БД:** <br/>

```shell
`openat`(AT_FDCWD, "/etc/magic", O_RDONLY) = 3

stat("/root/.magic.mgc", 0x7ffc52ffe2d0) = -1 ENOENT (No such file or directory)
stat("/root/.magic", 0x7ffc52ffe2d0)    = -1 ENOENT (No such file or directory)
`openat`(AT_FDCWD, "/etc/magic.mgc", O_RDONLY) = -1 ENOENT (No such file or directory)
stat("/etc/magic", {st_mode=S_IFREG|0644, st_size=111, ...}) = 0 <br/> <br/>
```

**3.** Предположим, приложение пишет лог в текстовый файл. <br/>
Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. <br/> 
Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. <br/> 
Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе). <br/><br/>


Нашёл большущий файл на много гигабайт, удалил его `rm ./testbig.log`

В итоге этот файл всё ещё открыт каким-то процессом и имеется открытый дескриптор этого файла и поэтому файл просто помечен как ***deleted***, но фактически не удалён. <br/>
Нужно Найти тот самый процесс и перезапустить его или закрыть дескриптор файла:
```shell
lsof +L1
или 
lsof | grep deleted

COMMAND  PID    USER   FD   TYPE DEVICE SIZE/OFF NLINK NODE NAME
top     2466 vagrant    0u   CHR  12136,1      0t0     0    4 /tmp/testbig.log (deleted)
```
Видим, что процесс с **PID=2466** имеет файловый дескриптор **4** <br/> 
Смотрим информацию по процессу `ps -p 2466` <br/>

Освобождаем место на диске:
убиваем процесс `kill -9 2466` или обнуляем файл  `sudo truncate -s 0 /proc/2466/fd/4 or > /proc/1366/fd/4` <br/> <br/>


**4.** Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)? <br/>

Зомби-процессы освобождают все свои ресурсы, но запись в таблице процессов остается. <br/><br/>


**5.** В `iovisor BCC` есть утилита `opensnoop`: <br/>
`root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop` <br/>
`/usr/sbin/opensnoop-bpfcc` <br/>
На какие файлы вы увидели вызовы группы open за первую секунду работы утилиты? <br/> 
Воспользуйтесь пакетом `bpfcc-tools` для Ubuntu 20.04. Дополнительные сведения по установке. <br/>


[open](https://disk.yandex.ru/i/g-1Yj9jCItbGJg) <br/><br/>


**6.** Какой системный вызов использует uname -a? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в `/proc`, где можно узнать версию ядра и релиз ОС. <br/>


`uname()` <br/> 
`Part of the utsname information is also accessible  via  /proc/sys/kernel/{ostype, hostname, 
osrelease, version, domainname}`. <br/><br/>

**7.** Чем отличается последовательность команд через`;` и через `&&` в bash? Например:
```shell
root@netology1:~# test -d /tmp/some_dir; echo Hi
Hi
root@netology1:~# test -d /tmp/some_dir && echo Hi
root@netology1:~#
```

**;** - позволяет разместить две и более команд в одной и той же строке, выполнение команды происходит последовательно. <br/>
**&&** - Командная оболочка будет интерпретировать последовательность символов && как логический оператор "И". <br/> 
При использовании оператора && вторая команда будет исполняться только в том случае, если исполнение первой команды успешно завершится (будет возвращен нулевой код завершения). <br/>
использование **&&** вместе с **set -e** не имеет смысла, так как при ошибке скрипта, выполнение команд прекратиться. <br/><br/>


**8.** Из каких опций состоит режим `bash set -euxo pipefail` и почему его хорошо было бы использовать в сценариях? <br/>

 - **e** - прекращает выполнение скрипта если команда завершилась ошибкой, выводит в stderr строку с ошибкой <br/>
 - **u** - прекращает выполнение скрипта, если встретилась несуществующая переменная <br/>
 - **x** - выводит выполняемые команды в stdout перед выполнением <br/>
 - **о** - `pipefail` - прекращает выполнение скрипта, даже если одна из частей pipe завершилась ошибкой <br/>

При таком запуске скрипт получается безопасным, происходит автоматическая обработка ошибок. <br/><br/>



**9.** Используя `-o stat` для `ps`, определите, какой наиболее часто встречающийся статус у процессов в системе. <br/>
В `man ps` ознакомьтесь (/PROCESS STATE CODES) что значат дополнительные к основной заглавной буквы статуса процессов. <br/>
Его можно не учитывать при расчете (считать `S`, `Ss` или `Ssl` равнозначными). <br/><br/>



**Ss** - процесс, ожидающий завершения, лидер сессии <br/>
**R+** - процесс выполняется, фоновый процесс <br/>

**s** -   является лидером сессии <br/>
**+** -   находится в группе процессов переднего плана <br/>

**l** - является многопоточным <br/>
**R** - запущенный или доступный для выполнения (в очереди выполнения) <br/>
