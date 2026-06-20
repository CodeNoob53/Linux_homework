# Домашнє завдання №6. Bash-скрипт бекапу логів

**Виконав:** Федотов Олександр  
**Модуль:** Module 6  
**Варіант:** A — Скрипт бекапу логів

---

## Опис скрипту

Скрипт `backup.sh` виконує резервне копіювання директорії з логами у вигляді стисненого архіву `.tar.gz`. Підтримує перевірку аргументів, захист від паралельного запуску та звітування про результат.

**Запуск:**
```bash
./backup.sh /path/to/logs /path/to/backup
```

---

## Код скрипту

```bash
#!/bin/bash

# --- Перевірка аргументів ---
if [ "$#" -ne 2 ]; then
    echo "Usage: ./backup.sh <log_dir> <backup_dir>"
    exit 1
fi

LOG_DIR="$1"
BACKUP_DIR="$2"

if [ ! -d "$LOG_DIR" ]; then
    echo "Usage: ./backup.sh <log_dir> <backup_dir>"
    exit 1
fi

if [ ! -d "$BACKUP_DIR" ]; then
    echo "Usage: ./backup.sh <log_dir> <backup_dir>"
    exit 1
fi

# --- Захист від паралельного запуску ---
LOCK_FILE="/tmp/backup.lock"

if [ -f "$LOCK_FILE" ]; then
    echo "Backup already running"
    exit 1
fi

# Створюємо lock-файл, видаляємо його при виході (навіть при помилці)
touch "$LOCK_FILE"
trap "rm -f '$LOCK_FILE'" EXIT

# --- Створення архіву ---
TIMESTAMP=$(date +"%Y-%m-%d_%H-%M")
ARCHIVE_NAME="logs_backup_${TIMESTAMP}.tar.gz"
ARCHIVE_PATH="${BACKUP_DIR}/${ARCHIVE_NAME}"

tar -czf "$ARCHIVE_PATH" -C "$LOG_DIR" .

# --- Перевірка результату ---
if [ $? -ne 0 ]; then
    echo "Backup failed"
    exit 2
fi

echo "Backup created: $ARCHIVE_PATH"
```

---

## Пояснення коду

### 1. Перевірка аргументів

```bash
if [ "$#" -ne 2 ]; then ...
```

- `$#` — кількість переданих аргументів; якщо не дорівнює 2 — виводить підказку і завершується з кодом `1`
- `$1`, `$2` — перший і другий аргументи (директорія логів і директорія бекапу)
- `[ ! -d "$DIR" ]` — перевіряє що директорія існує; `-d` повертає true якщо шлях є директорією

### 2. Захист від паралельного запуску

```bash
LOCK_FILE="/tmp/backup.lock"
if [ -f "$LOCK_FILE" ]; then ...
touch "$LOCK_FILE"
trap "rm -f '$LOCK_FILE'" EXIT
```

- Lock-файл `/tmp/backup.lock` — якщо він існує, значить інший екземпляр скрипту вже запущений
- `touch` — створює порожній lock-файл на час роботи скрипту
- `trap ... EXIT` — гарантує видалення lock-файлу при будь-якому завершенні скрипту (успішному, помилковому або через `Ctrl+C`)

### 3. Створення архіву

```bash
TIMESTAMP=$(date +"%Y-%m-%d_%H-%M")
ARCHIVE_NAME="logs_backup_${TIMESTAMP}.tar.gz"
tar -czf "$ARCHIVE_PATH" -C "$LOG_DIR" .
```

- `date +"%Y-%m-%d_%H-%M"` — формує рядок з поточною датою і часом (наприклад `2026-06-20_09-57`)
- Ім'я архіву містить timestamp: `logs_backup_2026-06-20_09-57.tar.gz`
- `tar -czf` — створює стиснений архів: `-c` (create), `-z` (gzip), `-f` (file)
- `-C "$LOG_DIR" .` — архівує вміст директорії без включення абсолютного шляху

### 4. Перевірка результату

```bash
if [ $? -ne 0 ]; then
    echo "Backup failed"
    exit 2
fi
echo "Backup created: $ARCHIVE_PATH"
```

- `$?` — код завершення останньої команди; `0` означає успіх, будь-яке інше — помилку
- При невдачі виводить `Backup failed` і завершується з кодом `2`
- При успіху виводить повний шлях до створеного архіву

---

## Тестування

### Підготовка

```bash
nano ~/backup.sh        # створення скрипту
chmod +x ~/backup.sh    # надання прав на виконання
mkdir -p ~/test_logs ~/test_backup
echo "log entry 1" > ~/test_logs/app.log
echo "log entry 2" > ~/test_logs/error.log
```

![Скрипт в nano](assets/script_nano.avif)

![Підготовка директорій та файлів](assets/script_prepare.avif)

---

### Тест 1 — неправильна кількість аргументів

```bash
./backup.sh
./backup.sh /home/aleax/test_logs
```

**Результат:**
```
Usage: ./backup.sh <log_dir> <backup_dir>
Usage: ./backup.sh <log_dir> <backup_dir>
```

Скрипт коректно виводить підказку при відсутності одного або обох аргументів.

![Тест — неправильні аргументи](assets/test_wrong_args.avif)

---

### Тест 2 — успішне виконання

```bash
./backup.sh ~/test_logs ~/test_backup
ls ~/test_backup/
```

**Результат:**
```
logs_backup_2026-06-20_09-57.tar.gz
```

Архів створено з правильним іменем що містить дату і час.

![Тест — успішний бекап](assets/test_success.avif)

---

### Тест 3 — захист від паралельного запуску

```bash
touch /tmp/backup.lock
./backup.sh ~/test_logs ~/test_backup
rm /tmp/backup.lock
```

**Результат:**
```
Backup already running
```

Скрипт виявив lock-файл і завершився без виконання бекапу.

![Тест — lock-файл](assets/test_lock.avif)
