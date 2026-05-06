# Домашнє завдання №3. Процеси та моніторинг системи

**Виконав:** Федотов Олександр  
**Модуль:** Module 3  

---

## Завдання 1. Огляд активних процесів (2 бали)

### 1. Список усіх процесів системи

```bash
ps aux
```

**Результат:**
```
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.3  0.6 166488 11496 ?        Ss   14:34   0:06 /sbin/init sp
root           2  0.0  0.0      0     0 ?        S    14:34   0:00 [kthreadd]
...
aleax       4038  6.1  3.0 638292 52428 ?        Ssl  15:11   0:02 /usr/libexec/
aleax       4066  0.2  0.2  22504  5120 pts/0    Ss   15:11   0:00 bash
aleax       4089  0.2  0.2  24164  3712 pts/0    R+   15:12   0:00 ps aux
```

**Пояснення команди та стовпців:**
- `ps aux` — виводить знімок усіх процесів у системі на момент виконання:
  - `a` — показати процеси всіх користувачів
  - `u` — детальний формат (користувач, CPU, пам'ять)
  - `x` — включити процеси без терміналу (фонові сервіси)
- `PID` — унікальний ідентифікатор процесу
- `%CPU` / `%MEM` — відсоток використання процесора і пам'яті
- `STAT` — стан процесу: `S` (sleeping), `R` (running), `I` (idle), `Ss` (лідер сесії)
- `COMMAND` — команда, що запустила процес

![Завдання 1 — ps aux](assets/img/task1_ps_aux.avif)

---

### 2. Встановлення та запуск htop

```bash
htop
# якщо не встановлений:
sudo apt install htop
```

**Процес що споживає найбільше RAM:** `/usr/bin/gnome-shell` — `10.0% MEM` (168M RSS)

**Пояснення:**
- `htop` — інтерактивний моніторинг процесів у реальному часі
- Список відсортовано за `MEM%` (F6 → MEM%) — вгорі процеси з найбільшим споживанням пам'яті
- `gnome-shell` — графічна оболонка робочого столу GNOME, займає найбільше RAM оскільки керує всім UI
- Верхня панель показує завантаження CPU (0, 1), використання `Mem` (549M/1.66G) та `Swap` (241M/6.19G)

![Завдання 1 — htop install](assets/img/task1_htop_install.avif)

![Завдання 1 — htop](assets/img/task1_htop.avif)

---

### 3. PID поточної оболонки

```bash
echo $$
```

**Результат:**
```
4066
```

**Пояснення:**
- `$$` — спеціальна змінна bash, що містить PID поточного процесу оболонки
- PID `4066` відповідає процесу `bash` видному у виводі `ps aux` вище

![Завдання 1 — PID](assets/img/task1_pid.avif)

---

## Завдання 2. Робота у фоні та керування процесами (2 бали)

```bash
sleep 1000 &
jobs
fg %1
# Ctrl+Z — зупинити
kill -9 4681
nohup sleep 500 &
jobs
```

**Результат:**
```
[1] 4681
[1]+  Running    sleep 1000 &

# fg %1 — повернуто на передній план
sleep 1000
^Z
[1]+  Stopped    sleep 1000

# після kill
[1]   Killed     sleep 1000

# nohup
[2] 4699
nohup: ignoring input and appending output to 'nohup.out'
[2]+  Running    nohup sleep 500 &
```

**Пояснення команд:**
- `sleep 1000 &` — запускає команду у фоновому режимі (`&`); термінал одразу повертає PID процесу
- `jobs` — показує список фонових завдань поточної сесії з їх номерами (`[1]`) та статусом
- `fg %1` — повертає завдання №1 з фону на передній план (`fg` — foreground)
- `Ctrl+Z` — надсилає сигнал `SIGTSTP`, зупиняє процес (не завершує, а "заморожує")
- `kill -9 4681` — примусово завершує процес за PID; `-9` це сигнал `SIGKILL` який не можна ігнорувати
- `nohup sleep 500 &` — запускає команду так, щоб вона продовжувала працювати після закриття терміналу; вивід перенаправляється у файл `nohup.out`

![Завдання 2](assets/img/task2_bg.avif)

---

## Завдання 3. Пріоритети та обмеження (4 бали)

```bash
nice -n 15 sleep 500 &
sudo renice -n 10 -p 1957
ulimit -a
```

**Результат `renice`:**
```
1957 (process ID) old priority 15, new priority 10
```

**Результат `ulimit -a`:**
```
real-time non-blocking time  (microseconds, -R) unlimited
core file size               (blocks, -c) 0
data seg size                (kbytes, -d) unlimited
scheduling priority                  (-e) 0
file size                    (blocks, -f) unlimited
pending signals                      (-i) 6475
max locked memory            (kbytes, -l) 216952
max memory size              (kbytes, -m) unlimited
open files                           (-n) 1024
pipe size                  (512 bytes, -p) 8
POSIX message queues          (bytes, -q) 819200
real-time priority                   (-r) 0
stack size                   (kbytes, -s) 8192
cpu time                   (seconds, -t) unlimited
max user processes                   (-u) 6475
virtual memory               (kbytes, -v) unlimited
file locks                           (-x) unlimited
```

**Пояснення команд:**
- `nice -n 15 sleep 500 &` — запускає команду з заданим пріоритетом:
  - значення `nice` від `-20` (найвищий пріоритет) до `19` (найнижчий)
  - `15` — низький пріоритет, ядро буде виділяти цьому процесу менше CPU
  - звичайний користувач може лише підвищувати значення (знижувати пріоритет)
- `sudo renice -n 10 -p 1957` — змінює пріоритет вже запущеного процесу:
  - `-n 10` — нове значення nice
  - `-p 1957` — PID процесу
  - при першій спробі без `sudo` команда повернула `Permission denied` — звичайний користувач не може підвищувати пріоритет (зменшувати значення nice) навіть для власного процесу; це захист ядра від того, щоб користувачі не захоплювали CPU в обхід планувальника
  - `sudo` надає права суперкористувача і дозволяє змінити пріоритет у будь-який бік
- `ulimit -a` — показує поточні обмеження ресурсів для користувача:
  - `open files 1024` — максимум відкритих файлів одночасно
  - `max user processes 6475` — максимум процесів користувача
  - `stack size 8192` — розмір стеку для процесів (kbytes)
  - `unlimited` — без обмежень з боку системи

![Завдання 3](assets/img/task3_nice.avif)

---

## Завдання 4. Моніторинг ресурсів (2 бали)

```bash
df -h
free -h
```

**Результат `df -h`:**
```
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           170M  1,9M  168M   2% /run
/dev/sda3       457G   16G  419G   4% /
tmpfs           848M     0  848M   0% /dev/shm
tmpfs           5,0M  4,0K  5,0M   1% /run/lock
/dev/sda2       512M  6,1M  506M   2% /boot/efi
tmpfs           170M  112K  170M   1% /run/user/1000
```

**Результат `free -h`:**
```
               total        used        free      shared  buff/cache   available
Mem:           1,7Gi       518Mi       293Mi       9,0Mi       882Mi       1,0Gi
Swap:          6,2Gi       240Mi       6,0Gi
```

**Пояснення команд:**
- `df -h` — показує використання дискового простору (`-h` — human-readable, тобто в MB/GB):
  - `/dev/sda3` — основний розділ, 457G всього, використано лише 16G (4%)
  - `tmpfs` — віртуальні файлові системи в RAM (не займають місця на диску)
  - `/boot/efi` — розділ завантажувача EFI
- `free -h` — показує стан оперативної пам'яті:
  - `Mem total 1,7Gi` — всього RAM
  - `used 518Mi` — зайнято процесами
  - `buff/cache 882Mi` — зайнято кешем (ядро автоматично звільняє при потребі)
  - `available 1,0Gi` — реально доступно для нових процесів
  - `Swap 6,2Gi` — розділ підкачки, використовується коли RAM заповнена

![Завдання 4](assets/img/task4_df_free.avif)
