# Домашнє завдання №5. Мережа, SSH та передача файлів

**Виконав:** Федотов Олександр  
**Модуль:** Module 5  

---

## Завдання 1. Мережева діагностика (2 бали)

### 1. IP-адреси та інтерфейси

```bash
ip a
```

**Результат (основні інтерфейси):**
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536
    inet 127.0.0.1/8 scope host lo
2: enp6s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> state DOWN
3: wlp7s0b1: <NO-CARRIER,BROADCAST,MULTICAST,UP> state DOWN
4: docker0: inet 172.17.0.1/16
6: enxf6b95c1662f8: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 10.153.131.123/24 scope global dynamic
```

**Пояснення:**
- `ip a` — виводить всі мережеві інтерфейси та їхні IP-адреси
- `lo` — loopback-інтерфейс (`127.0.0.1`), використовується для внутрішньої комунікації системи
- `enxf6b95c1662f8` — активний мережевий інтерфейс з локальною IP-адресою `10.153.131.123/24`
- `enp6s0` і `wlp7s0b1` — дротовий та Wi-Fi інтерфейси, наразі не підключені (`NO-CARRIER`)
- `docker0` — віртуальний міст Docker (`172.17.0.1`)

**Локальна IP-адреса:** `10.153.131.123`

![Завдання 1 — ip a](assets/task1_ip_a.avif)

---

### 2. Перевірка доступності інтернету

```bash
ping -c 4 8.8.8.8
```

**Результат:**
```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=116 time=178 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=116 time=54.8 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=116 time=55.3 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=116 time=51.9 ms

4 packets transmitted, 4 received, 0% packet loss, time 3004ms
```

**Пояснення:**
- `ping -c 4 8.8.8.8` — надсилає 4 ICMP-пакети на публічний DNS Google (`8.8.8.8`); `-c 4` обмежує кількість пакетів
- `0% packet loss` — всі пакети дійшли, інтернет-з'єднання є
- `ttl=116` — кількість хопів до вузла; `time` — затримка відповіді в мілісекундах

**Доступ до інтернету:** є ✓

![Завдання 1 — ping](assets/task1_ping.avif)

---

### 3. Відкриті listening-порти

```bash
ss -tulpn
```

**Результат:**
```
Netid  State   Local Address:Port
tcp    LISTEN  0.0.0.0:22        — SSH
tcp    LISTEN  [::]:22           — SSH (IPv6)
tcp    LISTEN  127.0.0.1:631     — CUPS (принтер)
tcp    LISTEN  127.0.0.1:35817
udp    UNCONN  0.0.0.0:5353      — mDNS
udp    UNCONN  0.0.0.0:631
```

**Пояснення:**
- `ss -tulpn` — показує всі відкриті сокети: `-t` TCP, `-u` UDP, `-l` listening, `-p` процес, `-n` числові адреси
- Порт `22` (`0.0.0.0:22`) — SSH-сервер слухає на всіх інтерфейсах
- Порт `631` — CUPS, служба друку
- Порт `5353` (UDP) — mDNS (Avahi), для локального виявлення пристроїв у мережі

**Сервіс що слухає порт:** SSH на порту 22

![Завдання 1 — ss -tulpn](assets/task1_ss_tulpn.avif)

---

## Завдання 2. SSH-доступ з ключами та config (4 бали)

### Підготовка — встановлення SSH-сервера

```bash
sudo apt install openssh-server
sudo systemctl status ssh
```

**Результат:**
```
● ssh.service - OpenBSD Secure Shell server
     Active: active (running) since Tue 2026-06-09 13:47:30 CEST
   Main PID: 3419 (sshd)
sshd[3419]: Server listening on 0.0.0.0 port 22.
```

**Пояснення:** OpenSSH-сервер встановлено і запущено. SSH демон (`sshd`) слухає підключення на порту 22. Завдання виконується локально — машина підключається сама до себе через `localhost`.

![Завдання 2 — ssh install status](assets/task2_ssh_install_status.avif)

![Завдання 2 — ssh status](assets/task2_ssh_status.avif)

---

### 1. Генерація SSH-ключа

```bash
ssh-keygen -t ed25519
```

**Результат:**
```
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/aleax/.ssh/id_ed25519):
Your identification has been saved in /home/aleax/.ssh/id_ed25519
Your public key has been saved in /home/aleax/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:b0yzPpnUIiyvKpPyUjHL4+Wlb5yDmw4KuvL6DJUJroU aleax@aleax-AO722
```

**Пояснення:**
- `ssh-keygen -t ed25519` — генерує пару ключів за алгоритмом Ed25519 (сучасний та надійний алгоритм)
- Створюються два файли: `id_ed25519` (приватний ключ) та `id_ed25519.pub` (публічний ключ)
- Приватний ключ зберігається локально і **ніколи не передається**; публічний копіюється на сервер
- Randomart image — графічне представлення відбитку ключа для візуальної перевірки

![Завдання 2 — ssh-keygen](assets/task2_ssh_keygen.avif)

---

### 2. Копіювання публічного ключа на сервер

```bash
ssh-copy-id aleax@localhost
```

**Результат:**
```
The authenticity of host 'localhost (127.0.0.1)' can't be established.
Are you sure you want to continue connecting (yes/no)? yes
Number of key(s) added: 1

Now try logging in with: "ssh 'aleax@localhost'"
```

**Пояснення:**
- `ssh-copy-id` — копіює публічний ключ у файл `~/.ssh/authorized_keys` на сервері
- При першому підключенні SSH запитує підтвердження відбитку хоста — відповідаємо `yes`
- Після цього система знає наш публічний ключ і більше не вимагатиме пароль

![Завдання 2 — ssh-copy-id](assets/task2_ssh_copy_id.avif)

---

### 3. Створення SSH config та підключення

```bash
nano ~/.ssh/config
```

**Вміст файлу:**
```
Host myserver
    HostName localhost
    User aleax
    IdentityFile ~/.ssh/id_ed25519
```

```bash
ssh myserver
```

**Результат:**
```
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-40-generic x86_64)
aleax@aleax-AO722: $
```

**Пояснення:**
- `~/.ssh/config` — конфігураційний файл SSH-клієнта; дозволяє задавати псевдоніми для серверів
- `Host myserver` — ім'я псевдоніму; замість `ssh aleax@localhost` можна просто писати `ssh myserver`
- `IdentityFile` — вказує який саме приватний ключ використовувати
- Підключення відбулось **без введення пароля** — автентифікація пройшла за ключем

**Host у config:** `myserver`  
**Підключення без пароля:** працює ✓

![Завдання 2 — config та підключення](assets/task2_ssh_config_connect.avif)

![Завдання 2 — exit](assets/task2_ssh_exit.avif)

![Завдання 2 — повторне підключення без пароля](assets/task2_ssh_no_password.avif)

---

## Завдання 3. Копіювання файлів між машинами (4 бали)

### 1. Створення тестового файлу

```bash
echo "test" > test.txt
cat test.txt
```

**Результат:**
```
test
```

**Пояснення:** `echo "test" > test.txt` — створює файл з вмістом "test"; `>` перенаправляє вивід у файл (перезаписує якщо існує).

![Завдання 3 — echo test](assets/task3_echo_test.avif)

---

### 2. Передача файлу через scp

```bash
scp test.txt aleax@localhost:~/
```

**Результат:**
```
test.txt    100%    5    0.8KB/s    00:00
```

**Пояснення:**
- `scp` (Secure Copy) — копіює файли між машинами через SSH-з'єднання
- Синтаксис: `scp <джерело> <користувач>@<хост>:<шлях>`
- `100%` — файл передано повністю; оскільки SSH вже налаштований з ключем — пароль не запитувався

**Шлях до файлу на сервері:** `/home/aleax/test.txt`

![Завдання 3 — scp](assets/task3_scp.avif)

---

### 3 & 4. Створення директорії та синхронізація через rsync

```bash
ssh myserver "mkdir -p ~/sync_dir"
mkdir -p ~/local_sync
echo "file1" > ~/local_sync/file1.txt
echo "file2" > ~/local_sync/file2.txt
rsync -av ~/local_sync/ aleax@localhost:~/sync_dir/
```

**Результат:**
```
sending incremental file list
./
file1.txt
file2.txt

sent 209 bytes  received 57 bytes  177,33 bytes/sec
total size is 12  speedup is 0,05
```

**Пояснення:**
- `ssh myserver "mkdir -p ~/sync_dir"` — виконує команду на сервері не заходячи в сесію; `-p` не виводить помилку якщо директорія вже є
- `rsync -av` — синхронізує директорії: `-a` (archive) зберігає права та атрибути, `-v` (verbose) показує що передається
- На відміну від `scp`, `rsync` передає лише **змінені файли** — ефективний для регулярної синхронізації
- `speedup is 0,05` — показник ефективності стиснення; при малих файлах він низький

**Шлях до файлів на сервері:** `/home/aleax/sync_dir/`

![Завдання 3 — rsync](assets/task3_rsync.avif)

---

### 5. Перевірка файлів через sftp

```bash
sftp aleax@localhost
```

**Команди в sftp:**
```
sftp> ls
sftp> ls sync_dir
sftp> exit
```

**Результат:**
```
Connected to localhost.
sftp> ls
lab2  local_sync  mylog.txt  myscript.sh  nohup.out  snap  sync_dir  test.txt  ...
sftp> ls sync_dir
sync_dir/file1.txt    sync_dir/file2.txt
```

**Пояснення:**
- `sftp` (SSH File Transfer Protocol) — інтерактивний файловий клієнт поверх SSH
- `ls` — виводить вміст поточної директорії на сервері; видно `test.txt` (переданий через `scp`) і `sync_dir` (синхронізований через `rsync`)
- `ls sync_dir` — перевіряємо вміст директорії: обидва файли `file1.txt` і `file2.txt` присутні

**Команда для перевірки:** `ls sync_dir`  
**Шлях до файлів:** `/home/aleax/sync_dir/file1.txt`, `/home/aleax/sync_dir/file2.txt`

![Завдання 3 — sftp](assets/task3_sftp.avif)
