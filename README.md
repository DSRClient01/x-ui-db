# 🚀 Оптимизация базы данных SQLite для `x-ui`

Этот скрипт выполняет **оптимизацию базы данных `x-ui`**, устраняя блокировки и улучшая производительность. После выполнения в папке `/etc/x-ui/` появятся дополнительные файлы, и это **нормальное поведение**.

## 📌 Как использовать?

1. **Сохраните скрипт** в файл, например, `x-ui_optimize_db.py`.
2. **Запустите его с правами `root`**:
   ```bash
   sudo python3 x_ui_optimize_db.py
   ```

## 🚀 Что делает скрипт?

### 1️⃣ Останавливает сервис `x-ui`
```bash
systemctl stop x-ui
```
Это нужно, чтобы избежать блокировки базы во время оптимизации.

### 2️⃣ Создаёт резервную копию базы в `/tmp/`
```bash
cp /etc/x-ui/x-ui.db /tmp/x-ui-<timestamp>.db
```
Файл резервной копии получает временную метку, например:
```
/tmp/x-ui-1711045600.db
```
Это позволяет откатить изменения в случае ошибки.

### 3️⃣ Оптимизирует базу данных с помощью SQL-команд:

| **Команда SQL** | **Описание** |
|---------------|-------------|
| `PRAGMA journal_mode=WAL;` | Включает режим WAL (Write-Ahead Logging), который позволяет **одновременно читать и записывать данные**. Это уменьшает ошибки `database locked`. |
| `PRAGMA wal_checkpoint(TRUNCATE);` | Очищает старые WAL-файлы, снижая нагрузку на диск. |
| `PRAGMA optimize;` | Оптимизирует индексы и кэш базы, ускоряя работу запросов. |
| `PRAGMA busy_timeout = 5000;` | Увеличивает время ожидания перед ошибкой `database locked` (5000 мс = 5 секунд). |
| `PRAGMA read_uncommitted = 1;` | Позволяет читать данные без ожидания завершения других транзакций (но может отдать **неподтверждённые** данные). |

### 4️⃣ Перезапускает `x-ui` после оптимизации
```bash
systemctl restart x-ui
```
После этого база будет работать быстрее и стабильнее.

## 📂 Почему появились файлы `x-ui.db-shm` и `x-ui.db-wal`?

После включения WAL в папке `/etc/x-ui/` появляются **дополнительные файлы**:

| **Файл** | **Описание** |
|----------|-------------|
| `x-ui.db` | Основной файл базы SQLite. |
| `x-ui.db-wal` | Журнал WAL (Write-Ahead Logging), в который сначала записываются изменения перед записью в основной файл. |
| `x-ui.db-shm` | Файл для управления совместным доступом к данным (Shared Memory). |

**Эти файлы создаются автоматически и не являются ошибкой.**

📌 **Важно:** Если `x-ui.db-wal` становится слишком большим, можно принудительно очистить WAL командой:
```sql
PRAGMA wal_checkpoint(TRUNCATE);
```

## ✅ Что делать, если что-то пошло не так?

1. Остановить сервис:
   ```bash
   systemctl stop x-ui
   ```  
2. Восстановить базу из бэкапа:
   ```bash
   cp /tmp/x-ui-<timestamp>.db /etc/x-ui/x-ui.db
   ```  
3. Запустить сервис:
   ```bash
   systemctl start x-ui
   ```
## Контакты

Telegram: [@DSRClient](https://t.me/DSRCLIENT)
