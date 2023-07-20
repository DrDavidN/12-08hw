# 12.08 «Резервное копирование баз данных» - Дрибноход Давид

### Задание 1. Резервное копирование

### Кейс
Финансовая компания решила увеличить надёжность работы баз данных и их резервного копирования. 

Необходимо описать, какие варианты резервного копирования подходят в случаях: 

1.1. Необходимо восстанавливать данные в полном объёме за предыдущий день.

1.2. Необходимо восстанавливать данные за час до предполагаемой поломки.

1.3.* Возможен ли кейс, когда при поломке базы происходило моментальное переключение на работающую или починенную базу данных.

*Приведите ответ в свободной форме.*
1.1 Подойдет метод дифференциального копирования, полный бэкап делается каждое воскресенье, а остальные дни пишутся диференциальные копии.  
1.2 К предыдущей стратегии добавить инкрементное копирование каждый час.  
1.3 Возможно использование репликации, когда кроме «главного» сервера («Мастера») постоянно работают ведомые сервера («слейвы»), которые получают данные с мастера в режиме реального времени.

---

### Задание 2. PostgreSQL

2.1. С помощью официальной документации приведите пример команды резервирования данных и восстановления БД (pgdump/pgrestore).

Создаём дамп
```sql
pg_dump -U user > /tmp/my.dump
```
Восстановление
```sql
pg_restore -d mydb my.dump

2.1.* Возможно ли автоматизировать этот процесс? Если да, то как?

Скрипт:
```bash
#!/bin/sh
PATH=/etc:/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin

PGPASSWORD=password
export PGPASSWORD
pathB=/backup
dbUser=dbuser
database=db

find $pathB \( -name "*-1[^5].*" -o -name "*-[023]?.*" \) -ctime +61 -delete
pg_dump -U $dbUser $database | gzip > $pathB/pgsql_$(date "+%Y-%m-%d").sql.gz

unset PGPASSWORD
```
Добавляем в расписание crontab. Запуск скрипта будет ежедневно в 1:00
```bash
1 0 * * * /scripts/postgresql_dump.sh
```

*Приведите ответ в свободной форме.*

---

### Задание 3. MySQL

3.1. С помощью официальной документации приведите пример команды инкрементного резервного копирования базы данных MySQL. 

В mysql enterprise edition 

```bash
mysqlbackup --defaults-file=/home/dbadmin/my.cnf \
  --incremental --incremental-base=history:last_backup \
  --backup-dir=/home/dbadmin/temp_dir \
  --backup-image=incremental_image1.bi \
   backup-to-image
```

С помощью утилиты XtraBackup
```bash
xtrabackup --backup --target-dir=/data/backups/inc1 --incremental-basedir=/data/backups/base
```

*Приведите ответ в свободной форме.*

---

Задания, помеченные звёздочкой, — дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.
