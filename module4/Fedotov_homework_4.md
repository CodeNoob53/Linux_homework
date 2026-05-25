# Домашнє завдання №4. Пакети, сервіси та журнали

**Виконав:** Федотов Олександр  
**Модуль:** Module 4  

---

## Завдання 1. Менеджери пакетів (2 бали)

### 1. Оновлення списку пакетів

```bash
sudo apt update
```

**Результат:**
```
Hit:1 http://ua.archive.ubuntu.com/ubuntu jammy InRelease
Get:2 http://security.ubuntu.com/ubuntu jammy-security InRelease [129 kB]
...
Fetched 3 872 kB in 5s
Reading package lists... Done
```

**Пояснення:**
- `apt update` — завантажує актуальні індекси пакетів з репозиторіїв; **не встановлює** жодного пакету, лише оновлює локальну базу даних доступних версій
- Після цього система знає, які нові версії з'явились у репозиторіях

![Завдання 1 — apt update](assets/task1_apt_update.avif)

---

### 2. Встановлення пакету tree

```bash
sudo apt install tree
```

**Результат:**
```
НОВІ пакунки, які будуть встановлені:
  tree
оновлено 0, встановлено 1 нових, 0 відмічено для видалення і 404 не оновлено.
Необхідно завантажити 47,9 kB архівів.
Після цієї операції об'єм зайнятого дискового простору зросте на 116 kB.
Отр:1 http://es.archive.ubuntu.com/ubuntu jammy/universe amd64 tree amd64 2.0.2-1 [47,9 kB]
Setting up tree (2.0.2-1) ...
```

**Пояснення:**
- `apt install tree` — встановлює пакет `tree` разом з усіма залежностями
- APT автоматично розраховує залежності, завантажує і встановлює пакет
- Перед встановленням показує список змін та запитує підтвердження

![Завдання 1 — apt install tree](assets/task1_apt_install_tree.avif)

---

### 3. Перевірка встановленого пакету та версія

```bash
dpkg -l tree
tree --version
```

**Результат `dpkg -l tree`:**
```
|| Name    Version    Architecture  Description
ii   tree   2.0.2-1    amd64         displays an indented directory tree, in color
```

**Результат `tree --version`:**
```
tree v2.0.2 (c) 1996 - 2022 by Steve Baker, Thomas Moore, Francesc Rocher, Florian Sesser, Kyosuke Tokoro
```

**Пояснення:**
- `dpkg -l tree` — перевіряє статус пакету в базі dpkg:
  - `ii` — пакет встановлений і працює коректно (перша `i` — бажаний стан "installed", друга `i` — фактичний стан "installed")
  - показує версію `2.0.2-1` та архітектуру `amd64`
- `tree --version` — викликає саму утиліту і виводить її версію

![Завдання 1 — dpkg та version](assets/task1_tree_version.avif)

---

### 4. Видалення пакету

```bash
sudo apt remove tree
dpkg -l tree
```

**Результат:**
```
Пакунки, які будуть ВИДАЛЕНІ:
  tree
Removing tree (2.0.2-1) ...

dpkg-query: no packages found matching tree
```

**Пояснення:**
- `apt remove tree` — видаляє бінарні файли пакету, але залишає конфігураційні файли (якщо вони є)
- `dpkg -l tree` після видалення повертає `no packages found` — пакет повністю відсутній у системі
- Для повного видалення разом з конфігами використовується `apt purge tree`

![Завдання 1 — apt remove](assets/task1_apt_remove_tree.avif)

---

## Завдання 2. Керування сервісами через systemctl (2 бали)

> **Примітка:** Спочатку було перевірено сервіс `ssh`, однак система повернула `Unit ssh.service could not be found` — OpenSSH не встановлений за замовчуванням у цій конфігурації Ubuntu. Відповідно до умов завдання ("або cron, або nginx, якщо є") було обрано сервіс `cron`, який присутній у системі.

```bash
sudo systemctl status ssh
# Unit ssh.service could not be found
systemctl list-units --type=service --state=running
```

![Завдання 2 — ssh not found, список сервісів](assets/task2_ssh_not_found.avif)

---

### 1. Статус сервісу cron

```bash
sudo systemctl status cron
```

**Результат:**
```
● cron.service - Regular background program processing daemon
     Loaded: loaded (/lib/systemd/system/cron.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2026-05-25 12:35:42 CEST; 4h 34min ago
   Main PID: 690 (cron)
```

**Пояснення:**
- `systemctl status <сервіс>` — показує поточний стан сервісу: чи запущений, скільки часу працює, PID процесу та останні записи журналу
- `active (running)` — сервіс працює нормально
- `enabled` — сервіс додано в автозавантаження

![Завдання 2 — cron status active](assets/task2_cron_status_active.avif)

---

### 2. Зупинка сервісу

```bash
sudo systemctl stop cron
sudo systemctl status cron
```

**Результат:**
```
○ cron.service - Regular background program processing daemon
     Active: inactive (dead) since Mon 2026-05-25 17:11:22 CEST; 18s ago
     ...
systemd[1]: Stopped Regular background program processing daemon.
```

**Пояснення:**
- `systemctl stop` — надсилає сервісу сигнал завершення роботи (SIGTERM)
- `inactive (dead)` — сервіс зупинений, процес не запущений
- В журналі видно запис `Stopped Regular background program processing daemon.`

![Завдання 2 — cron stop inactive](assets/task2_cron_stop_inactive.avif)

---

### 3. Повторний запуск сервісу

```bash
sudo systemctl start cron
sudo systemctl status cron
```

**Результат:**
```
● cron.service - Regular background program processing daemon
     Active: active (running) since Mon 2026-05-25 17:17:22 CEST; 16s ago
   Main PID: 5397 (cron)
systemd[1]: Started Regular background program processing daemon.
```

**Пояснення:**
- `systemctl start` — запускає сервіс; новий процес отримує новий PID (`5397` замість попереднього `690`)
- В журналі з'являється запис `Started Regular background program processing daemon.`

![Завдання 2 — cron start active](assets/task2_cron_start_active.avif)

---

### 4. Додавання в автозавантаження

```bash
sudo systemctl enable cron
sudo systemctl is-enabled cron
```

**Результат:**
```
Synchronizing state of cron.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable cron
enabled
```

**Пояснення:**
- `systemctl enable` — створює символічні посилання в `/etc/systemd/system/`, щоб сервіс запускався автоматично при завантаженні системи
- `systemctl is-enabled` — перевіряє чи увімкнено автозапуск; виводить `enabled`
- Різниця між `start` і `enable`: `start` запускає зараз, `enable` налаштовує автозапуск після перезавантаження

![Завдання 2 — cron enable](assets/task2_cron_enable.avif)

---

## Завдання 3. Робота з логами (2 бали)

### 1. Останні 10 рядків syslog

```bash
cd /var/log
sudo tail -n 10 syslog
```

**Результат:**
```
May 25 17:19:43 aleax-AO722 systemd[1]: Configuration file /run/systemd/system/netplan-ovs-cleanup.service is marked world-inaccessible.
May 25 17:19:45 aleax-AO722 systemd[1]: Reloading.
May 25 17:20:25 aleax-AO722 gnome-shell[1120]: g_object_get: assertion 'G_IS_OBJECT (object)' failed
May 25 17:20:28 aleax-AO722 systemd[916]: Starting Tracker metadata extractor...
May 25 17:20:29 aleax-AO722 systemd[916]: Started Tracker metadata extractor.
May 25 17:21:17 aleax-AO722 systemd[916]: Started VTE child process 5521 launched by gnome-terminal-server process 4937.
May 25 17:21:59 aleax-AO722 systemd[916]: Started VTE child process 5531 launched by gnome-terminal-server process 4937.
```

**Пояснення:**
- `tail -n 10` — виводить останні 10 рядків файлу; корисно для швидкого перегляду свіжих подій
- `/var/log/syslog` — головний системний журнал Ubuntu, куди пишуть більшість системних служб
- Кожен рядок містить: дату/час, ім'я хоста, назву процесу і PID, саме повідомлення

![Завдання 3 — syslog tail](assets/task3_syslog_tail.avif)

---

### 2. Перегляд помилок через journalctl

```bash
sudo journalctl -p err --no-pager | tail -n 20
```

**Результат:**
```
may 25 17:13:12 aleax-AO722 kernel: device offline error, dev sdc, sector 21856704 op 0x1:(WRITE)
may 25 17:13:12 aleax-AO722 kernel: Buffer I/O error on dev sdc3, logical block 26680, lost async page write
may 25 17:13:12 aleax-AO722 kernel: ntfs3: sdc1: failed to read volume at offset 0x24000
...
```

**Пояснення:**
- `journalctl -p err` — фільтрує журнал за рівнем пріоритету `err` (помилки) і вище
- `--no-pager` — виводить результат одразу в термінал без пагінації
- Рівні пріоритету journalctl: `emerg`, `alert`, `crit`, `err`, `warning`, `notice`, `info`, `debug` — вказуючи рівень, отримуємо всі повідомлення цього рівня і вище

![Завдання 3 — journalctl err](assets/task3_journalctl_err.avif)

---

### 3. Пошук записів про роботу cron

```bash
sudo journalctl -u cron --no-pager | tail -n 20
```

**Результат:**
```
may 25 17:11:22 aleax-AO722 systemd[1]: Stopping Regular background program processing daemon...
may 25 17:11:22 aleax-AO722 systemd[1]: cron.service: Deactivated successfully.
may 25 17:11:22 aleax-AO722 systemd[1]: Stopped Regular background program processing daemon.
may 25 17:17:22 aleax-AO722 systemd[1]: Started Regular background program processing daemon.
may 25 17:17:22 aleax-AO722 cron[5397]: (CRON) INFO (pidfile fd = 3)
may 25 17:17:22 aleax-AO722 cron[5397]: (CRON) INFO (Skipping @reboot jobs -- not system startup)
```

**Пояснення:**
- `journalctl -u cron` — показує всі записи журналу, що стосуються конкретного сервісу (`-u` — unit)
- Чітко видно події із Завдання 2: зупинка о `17:11:22` (`Stopped`) і запуск о `17:17:22` (`Started`)
- `Skipping @reboot jobs` — cron повідомляє що пропускає задачі позначені `@reboot`, бо це не системне завантаження, а ручний старт

![Завдання 3 — journalctl cron](assets/task3_journalctl_cron.avif)

---

## Завдання 4. Створення власного сервісу (4 бали)

### 1. Bash-скрипт

```bash
nano ~/myscript.sh
```

**Вміст скрипту:**
```bash
#!/bin/bash
while true; do
    echo "$(date)" >> ~/mylog.txt
    sleep 1
done
```

```bash
chmod +x ~/myscript.sh
```

**Пояснення:**
- `#!/bin/bash` — шебанг-рядок, вказує інтерпретатор для виконання скрипту
- `while true; do ... done` — нескінченний цикл
- `echo "$(date)" >> ~/mylog.txt` — записує поточну дату і час у файл `mylog.txt` (`>>` — дозапис без перезапису)
- `sleep 1` — пауза 1 секунда між записами
- `chmod +x` — надає скрипту право на виконання

![Завдання 4 — nano myscript.sh](assets/task4_nano_myscript.avif)

![Завдання 4 — chmod myscript](assets/task4_chmod_myscript.avif)

---

### 2. Unit-файл сервісу

```bash
sudo nano /etc/systemd/system/myscript.service
```

**Вміст файлу:**
```ini
[Unit]
Description=My Script Service

[Service]
ExecStart=/bin/bash /home/aleax/myscript.sh
User=aleax
Restart=always

[Install]
WantedBy=multi-user.target
```

**Пояснення секцій unit-файлу:**
- `[Unit]` — метадані: `Description` відображається у `systemctl status` та журналах
- `[Service]` — параметри запуску:
  - `ExecStart=/bin/bash /home/aleax/myscript.sh` — повний шлях до інтерпретатора і скрипту; systemd запускає сервіси без shell-оточення тому потрібен абсолютний шлях
  - `User=aleax` — сервіс запускається від імені користувача `aleax`, а не `root`; без цього параметра `~` розгортається як `/root/` і файл пишеться туди
  - `Restart=always` — systemd автоматично перезапускає сервіс якщо він завершується
- `[Install]` — `WantedBy=multi-user.target` означає що сервіс стартує при звичайному завантаженні системи (після команди `systemctl enable`)

![Завдання 4 — nano myscript.service](assets/task4_nano_service.avif)

---

### 3. Запуск та перевірка сервісу

```bash
sudo systemctl daemon-reload
sudo systemctl start myscript
sudo systemctl status myscript
```

**Результат:**
```
● myscript.service - My Script Service
     Loaded: loaded (/etc/systemd/system/myscript.service; disabled; vendor preset: enabled)
     Active: active (running) since Mon 2026-05-25 18:39:32 CEST; 38s ago
   Main PID: 10713 (bash)
     CGroup: /system.slice/myscript.service
             ├─10713 /bin/bash /home/aleax/myscript.sh
             └─10869 sleep 1
systemd[1]: Started My Script Service.
```

**Пояснення:**
- `daemon-reload` — перечитує всі unit-файли; обов'язково виконувати після створення або зміни `.service` файлу
- `Active: active (running)` — сервіс успішно запущений
- У `CGroup` видно два процеси: основний bash-скрипт (PID 10713) і поточний `sleep 1` (PID 10869) — підтвердження що цикл працює

![Завдання 4 — myscript status](assets/task4_myscript_status.avif)

---

### 4. Перевірка запису даних у файл

```bash
sleep 5
cat ~/mylog.txt
```

**Результат:**
```
lun 25 may 2026 18:39:32 CEST
lun 25 may 2026 18:39:33 CEST
lun 25 may 2026 18:39:34 CEST
lun 25 may 2026 18:39:35 CEST
...
lun 25 may 2026 18:40:04 CEST
```

**Пояснення:**
- `sleep 5` — чекаємо 5 секунд щоб скрипт встиг записати кілька рядків
- `cat ~/mylog.txt` — виводить вміст файлу; видно що дата записується щосекунди, починаючи з моменту запуску сервісу `18:39:32`
- Файл знаходиться в `/home/aleax/mylog.txt` завдяки параметру `User=aleax` в unit-файлі

![Завдання 4 — mylog cat](assets/task4_mylog_cat.avif)
