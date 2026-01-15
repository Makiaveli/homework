## BASH

### Написать скрипт для CRON, который раз в час формирует отчёт и отправляет его на заданную почту.


### Отчёт должен содержать:
<ul>
  <li>IP-адреса с наибольшим числом запросов (с момента последнего запуска);</li>
  <li>Запрашиваемые URL с наибольшим числом запросов (с момента последнего запуска);</li>
  <li>Ошибки веб-сервера/приложения (с момента последнего запуска);</li>
  <li>HTTP-коды ответов с указанием их количества (с момента последнего запуска).</li>
  <li>Скрипт должен предотвращать одновременный запуск нескольких копий, до его завершения.</li>
  <li>В письме должен быть прописан обрабатываемый временной диапазон.</li>
</ul>

### Создаем директорию, разрешаем выполнять скрипт

```bash
hwuser@hwlab:~$ sudo mkdir -p /var/lib/apache-report
hwuser@hwlab:~$ sudo chown $USER:$USER /var/lib/apache-report
hwuser@hwlab:~$ sudo chmod +x /usr/local/bin/apache-hourly-report.sh
```

### Скрипт

```bash
#!/usr/bin/env bash

LOCKFILE="/var/lib/apache-report/report.lock"
LASTRUN="/var/lib/apache-report/last_run"

REPORT_MAIL_TO="admin@adminimum.ru"

exec 9>"$LOCKFILE"
if ! flock -n 9 ; then
  exit 0
fi

NOW=$(date +"%Y-%m-%d %H:%M:%S")

if [ -f "$LASTRUN" ]; then
    LAST=$(cat "$LASTRUN")
else
    LAST=$(date -d "1 hour ago" +"%Y-%m-%d %H:%M:%S")
fi

TMP=$(mktemp)


LOGDATA=$(mktemp)
zgrep -h "" /var/log/apache2/access.log* | \
awk -v last="$LAST" -v now="$NOW" '$4 > "["last && $4 < "["now' > "$LOGDATA"

{
echo "Отчёт по Apache"
echo "Диапазон: c $LAST по $NOW"
echo

echo "------ TOP IP ------"
awk '{print $1}' "$LOGDATA" | sort | uniq -c | sort -nr | head

echo
echo "------ TOP URL ------"
awk '{print $7}' "$LOGDATA" | sort | uniq -c | sort -nr | head

echo
echo "------ HTTP CODES ------"
awk '{print $9}' "$LOGDATA" | sort | uniq -c | sort -nr

echo
echo "------ ERRORS (последние 50 строк в диапазоне)------"
zgrep -h "" /var/log/apache2/error.log* | sed -n "/$LAST/,/$NOW/p" | tail -50

} > "$TMP"

mail -s "Apache hourly report $NOW" "$REPORT_MAIL_TO" < "$TMP"

rm "$TMP" "$LOGDATA"

echo "$NOW" > "$LASTRUN"

```
### Добавляем задание в cron
```bash
hwuser@hwlab:~$  crontab -e
```
```bash
0 * * * * /usr/local/bin/apache-hourly-report.sh
```
