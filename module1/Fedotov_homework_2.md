# Домашнє завдання №2. Файлова система і права доступу

**Виконав:** Федотов Олександр  
**Модуль:** Module 1  

---

## Завдання 1. Ієрархія каталогів Linux (1 бал)

### 1. Перейти в кореневий каталог / і показати вміст

```bash
cd /
ls
```

**Результат:**
```
bin   dev  lib    libx32      mnt   root  snap      sys  var
boot  etc  lib32  lost+found  opt   run   srv       tmp
cdrom home lib64  media       proc  sbin  swapfile  usr
```

**Пояснення:**
- `cd /` — перехід у кореневий каталог, з якого починається вся файлова система Linux (абсолютний шлях)
- `ls` — виводить вміст поточного каталогу
- Видно ключові каталоги FHS: `bin` (команди), `etc` (конфіги), `home` (користувачі), `usr` (програми), `var` (змінні дані), `proc`/`sys` (віртуальні файлові системи)

![Завдання 1 — кореневий каталог](assets/img/m2/task1_root.avif)

---

### 2. Перейти в /etc і показати вміст

```bash
cd /etc
ls
```

**Результат:**
```
acpi                    cups             gshadow         legal           NetworkManager  rc6.d          systemd
adduser.conf            cupshelpers      gshadow-        libao.conf      networks        rcS.d          terminfo
alsa                    dbus-1           grub.d          libaudit.conf   newt            resolv.conf    thermald
alternatives            dconf            gss             libblockdev     nftables.conf   rmt            thunderbird
anacrontab              debconf.conf     gtk-2.0         libnl-3         nsswitch.conf   rpc            timezone
apg.conf                debian_version   gtk-3.0         libpaper.d      openvpn         rsyslog.conf   tmpfiles.d
apm                     default          hdparm.conf     libreoffice     opt             rsyslog.d      ubuntu-advantage
apparmor                deluser.conf     host.conf       locale.alias    os-release      rygel.conf     ucf.conf
...
```

**Пояснення:**
- `cd /etc` — перехід у каталог конфігурацій системи (абсолютний шлях)
- `/etc` містить конфігураційні файли всіх сервісів і програм: `hostname` (ім'я машини), `passwd` (користувачі), `hosts` (DNS), `fstab` (точки монтування), `ssh/` (налаштування SSH) тощо
- Це "мерія" системи — тут зберігаються всі налаштування, окремо від програмного коду

![Завдання 1 — /etc](assets/img/m2/task1_etc.avif)

---

### 3. Перейти у каталог /home і показати список користувачів

```bash
cd /home
ls
```

**Результат:**
```
aleax
```

**Пояснення:**
- `cd /home` — перехід у каталог домашніх директорій користувачів
- `ls` — виводить список папок, кожна з яких відповідає одному користувачу системи
- У даному випадку в системі один користувач — `aleax`, його домашня директорія `/home/aleax`

![Завдання 1 — /home](assets/img/m2/task1_home.avif)

---

## Завдання 2. Файли, каталоги та посилання (2 бали)

```bash
mkdir ~/lab2
echo "Hello, Linux!" > ~/lab2/file.txt
cat ~/lab2/file.txt
cp ~/lab2/file.txt ~/lab2/file_copy.txt
mv ~/lab2/file_copy.txt ~/lab2/file_renamed.txt
ln ~/lab2/file.txt ~/lab2/file_hard.txt
ln -s ~/lab2/file.txt ~/lab2/file_symlink.txt
find ~ -name "file.txt"
ls -la ~/lab2
```

**Пояснення команд:**
- `mkdir ~/lab2` — створює новий каталог `lab2` у домашній директорії (`~` — скорочення для `/home/aleax`)
- `echo "Hello, Linux!" > ~/lab2/file.txt` — записує текст у файл через перенаправлення `>` (створює файл якщо не існує)
- `cat ~/lab2/file.txt` — виводить вміст файлу в термінал
- `cp file.txt file_copy.txt` — копіює файл під новим іменем (`cp` — copy)
- `mv file_copy.txt file_renamed.txt` — перейменовує файл (`mv` — move, але без зміни каталогу працює як rename)
- `ln file.txt file_hard.txt` — створює **жорстке посилання**: два імені вказують на один і той самий inode на диску; якщо видалити оригінал — дані збережуться
- `ln -s file.txt file_symlink.txt` — створює **символічне посилання**: файл-ярлик що вказує на шлях до оригіналу; якщо оригінал видалити — посилання стане недійсним
- `find ~ -name "file.txt"` — рекурсивний пошук файлу за іменем починаючи з домашньої директорії
- `ls -la ~/lab2` — детальний список файлів (`-l` — довгий формат, `-a` — показати приховані)

**Результат `find`:**
```
/home/aleax/.local/share/Trash/files/lab2/file.txt
/home/aleax/lab2/file.txt
```

**Результат `ls -la ~/lab2`:**
```
total 20
drwxrwxr-x  2 aleax aleax 4096 abr 20 22:22 .
drwxr-x--- 19 aleax aleax 4096 abr 20 22:20 ..
-rw-rw-r--  2 aleax aleax   14 abr 20 22:20 file_hard.txt
-rw-rw-r--  1 aleax aleax   14 abr 20 22:21 file_renamed.txt
lrwxrwxrwx  1 aleax aleax   25 abr 20 22:22 file_symlink.txt -> /home/aleax/lab2/file.txt
-rw-rw-r--  2 aleax aleax   14 abr 20 22:20 file.txt
```

**Пояснення:**
- `file.txt` та `file_hard.txt` мають лічильник посилань `2` — це один і той самий inode (жорстке посилання)
- `file_symlink.txt` позначений `l` і вказує стрілкою `->` на оригінал — символічне посилання

![Завдання 2](assets/img/m2/task2_lab2.avif)

---

## Завдання 3. Права доступу (1 бал)

```bash
ls -l ~/lab2/file.txt
chmod 444 ~/lab2/file.txt
ls -l ~/lab2/file.txt
chmod u+w ~/lab2/file.txt
ls -l ~/lab2/file.txt
umask
umask 022
umask
```

**Результати:**
```
-rw-rw-r-- 2 aleax aleax 14 abr 20 22:20 /home/aleax/lab2/file.txt

# після chmod 444
-r--r--r-- 2 aleax aleax 14 abr 20 22:20 /home/aleax/lab2/file.txt

# після chmod u+w
-rw-r--r-- 2 aleax aleax 14 abr 20 22:20 /home/aleax/lab2/file.txt

# umask
0002

# після umask 022
0022
```

**Пояснення:**
- `chmod 444` — тільки читання для всіх (власник, група, інші)
- `chmod u+w` — додає право запису лише власнику
- `umask 0002` — дефолт Ubuntu, дозволяє групі писати у нові файли
- `umask 0022` — стандартне значення: нові файли матимуть права `644`, каталоги — `755`
  - `644` — власник може читати і писати (`rw-`), група і інші — тільки читати (`r--`)
  - `755` — власник може все (`rwx`), група і інші — читати і виконувати (`r-x`)

![Завдання 3](assets/img/m2/task3_permissions.avif)

---

## Завдання 4. Користувачі (1 бал)

```bash
sudo useradd -m -s /bin/bash trainee
sudo usermod -aG sudo trainee
grep trainee /etc/passwd
```

**Пояснення команд:**
- `sudo useradd -m -s /bin/bash trainee` — створює нового користувача:
  - `sudo` — виконати з правами суперкористувача (потрібні для зміни системних файлів)
  - `-m` — автоматично створити домашню директорію `/home/trainee`
  - `-s /bin/bash` — встановити bash як оболонку за замовчуванням (без цього Ubuntu встановить `/bin/sh`)
  - `trainee` — ім'я нового користувача
- `sudo usermod -aG sudo trainee` — додає користувача до групи `sudo`:
  - `-a` — append, додати до групи (без цього прапора замінить всі групи)
  - `-G sudo` — група до якої додати
- `grep trainee /etc/passwd` — шукає рядок з `trainee` у файлі `/etc/passwd`, де зберігається інформація про всіх користувачів системи

**Результат:**
```
trainee:x:1001:1001::/home/trainee:/bin/bash
```

**Пояснення полів:**
| Поле | Значення | Опис |
|---|---|---|
| `trainee` | trainee | Ім'я користувача |
| `x` | x | Пароль зберігається у `/etc/shadow` |
| `1001` | 1001 | UID — унікальний ідентифікатор користувача |
| `1001` | 1001 | GID — ідентифікатор основної групи |
| (порожньо) | | Поле GECOS (повне ім'я/коментар) |
| `/home/trainee` | /home/trainee | Домашній каталог |
| `/bin/bash` | /bin/bash | Оболонка за замовчуванням |

![Завдання 4](assets/img/m2/task4_user.avif)
