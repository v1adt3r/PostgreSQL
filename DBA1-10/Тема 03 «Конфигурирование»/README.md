# Конфигурирование

## Параметры
* Задача
  * управление работой и поведением СУБД
* Установка 
  * для экземпляра — файлы конфигурации
  * для отдельной базы или пользователя
  * для отдельного сеанса

В PostgreSQL существует большое количество параметров, влияющих на
работу СУБД. Параметры позволяют управлять потреблением ресурсов,
настраивать работу серверных процессов и многое другое.
Например, при помощи параметра max_connections можно ограничить
количество одновременных подключений к серверу.
Полный список и описание параметров конфигурации:
https://postgrespro.ru/docs/postgresql/10/runtime-config.html
В этой теме мы не изучаем назначение отдельных параметров
конфигурации, а лишь рассматриваем какими способами им можно
устанавливать значения.
Для установки параметров, в первую очередь, используются файлы
конфигурации. Если не определено иное, значения установленные в этих
файлах действуют для всего экземпляра СУБД.
Ряд параметров можно установить для отдельной базы данных или
пользователя. Такие установки будут иметь предпочтение перед файлами
конфигурации. Мы подробнее коснёмся этого варианта в следующих темах
курса.
Наконец, многими параметрами можно управлять на уровне отдельного
сеанса, прямо во время работы

## postgresql.conf

* Основной файл конфигурации
  * читается один раз при старте сервера
  * если параметр указан несколько раз, применяется последнее значение
* Расположение
  * SHOW config_file;
  * при сборке по умолчанию — в каталоге с данными (PGDATA)
* Действия при изменении
  * файл надо перечитать одним из способов:
  * `$ pg_ctl reload`
  * `$ kill -HUP`
  * `=> select pg_reload_conf();`
* изменения некоторых параметров требует перезапуска сервера

Основной конфигурационный файл — `postgresql.conf`.
Расположение файла задается при сборке PostgreSQL. Значение по
умолчанию — каталог с данными `(PGDATA)`, но пакетные дистрибутивы
обычно размещают этот файл в другом месте, в соответствии с правилами
принятыми в конкретной ОС.
Это текстовый, хорошо документированный, файл, хранящий параметры в
формате `«ключ=значение»`.
Если один и тот же параметр указан в конфигурационном файле (файлах)
несколько раз, то использоваться будет значение считанное последним.
Для вступления в силу внесенных в файл изменений, необходимо, чтобы
сервер перечитал файл. Для некоторых параметров требуется перезагрузка
сервера.

## postgresql.auto.conf

* Файл конфигурации, управляемый SQL
  * `ALTER SYSTEM SET параметр TO значение;` - добавляет или изменяет строку
  * `ALTER SYSTEM RESET параметр;` - удаляет строку
  * `ALTER SYSTEM RESET ALL;` - удаляет все строки
  * **считывается после `postgresql.conf`**
* Расположение
  * всегда в каталоге с данными (PGDATA)
* Действия при изменении
  * аналогично postgresql.conf

Самым последним считывается файл `postgresql.auto.conf`. Этот файл всегда
располагается в каталоге данных (`PGDATA`).
Этот файл не следует изменять вручную. Для его редактирования
предназначена команда `ALTER SYSTEM`. По сути `ALTER SYSTEM`
представляет собой SQL-интерфейс для удаленного управления
параметрами конфигурации.
Для применения изменений, сделанных `ALTER SYSTEM`, сервер должен
перечитать конфигурационные файлы, как и в случае с `postgresql.conf`.
Содержимое обоих файлов (`postgresql.conf` и `postgresql.auto.conf`) можно
увидеть через представление `pg_file_settings`.
А актуальные значения параметров — в представлении `pg_settings`.
Более подробная информация о команде `ALTER SYSTEM`:
https://postgrespro.ru/docs/postgresql/10/sql-altersystem.htm

## Практика

### Файл postgresql.conf и предаставление PG_FILE_SETTINGS
```shell
# Посмотрим небольшой фрагмент конфигурационного файла.

=> SELECT setting FROM pg_settings WHERE name = 'config_file';     
                setting                
---------------------------------------
 /usr/local/pgsql/data/postgresql.conf
(1 row)

=> \! sed -n '35,46p' /usr/local/pgsql/data/postgresql.conf 
#------------------------------------------------------------------------------
# FILE LOCATIONS
#------------------------------------------------------------------------------

# The default values of these variables are driven from the -D command-line
# option or PGDATA environment variable, represented here as ConfigDir.

#data_directory = 'ConfigDir'		# use data in another directory
					# (change requires restart)
#hba_file = 'ConfigDir/pg_hba.conf'	# host-based authentication file
					# (change requires restart)
#ident_file = 'ConfigDir/pg_ident.conf'	# ident configuration file

# Большинство строк в файле закомментированы, а для соответсвующих параметров 
# используются значения по умолчанию.
```

### Просмотр незадокументированых строк
```shell
# Чтобы увидеть все незакоментированные строки конфигурационного файла, можно
# обратится к представлению pg_file_settings:

=> SELECT sourceline, name, setting, applied FROM pg_file_settings WHERE sourcefile LIKE '%postgresql.conf%'; 
 sourceline |            name            |      setting       | applied 
------------+----------------------------+--------------------+---------
         65 | max_connections            | 100                | t
        127 | shared_buffers             | 128MB              | t
        150 | dynamic_shared_memory_type | posix              | t
        240 | max_wal_size               | 1GB                | t
        241 | min_wal_size               | 80MB               | t
        580 | log_timezone               | Europe/Moscow      | t
        694 | datestyle                  | iso, dmy           | t
        696 | timezone                   | Europe/Moscow      | t
        710 | lc_messages                | ru_RU.UTF-8        | t
        712 | lc_monetary                | ru_RU.UTF-8        | t
        713 | lc_numeric                 | ru_RU.UTF-8        | t
        714 | lc_time                    | ru_RU.UTF-8        | t
        717 | default_text_search_config | pg_catalog.russian | t
(13 rows)

# Столбец applied говорит о том, можно ли применить значение без перезапуска сервера. 
# Представление pg_file_settings показывает лишь содержимое файлов конфигурации,
# поэтому реальные значения параметров могут отличаться.
```

### Представление pg_settings
```shell
# Получить действующее значения всех параметров можно в представлении pg_settings.
# Вот что в нем содержится, например, для work_mem.

=> SELECT name, setting, unit, boot_val, reset_val,
    source, sourcefile, sourceline,
    pending_restart, context
    FROM pg_settings WHERE name = 'work_mem'\gx
-[ RECORD 1 ]---+---------
name            | work_mem
setting         | 4096
unit            | kB
boot_val        | 4096
reset_val       | 4096
source          | default
sourcefile      | 
sourceline      | 
pending_restart | f
context         | user

# name, setting, unit - название и значения параметра.
# boot_val - значение по умолчанию.
# reset_val - если параметр был изменен во время сеанса, то командой RESET можно восстановить
# это значение.
# source - источник текущего значения параметра.
# pending_restart - значение изменено в файле конфигурации, но требуется перезапуск сервера.

# Столбец context определяет действия, необходимые для применения параметра.
# Среди возможных значений:
# internal - изменить нельзя, задано при установке.
# postmaster - требуется перезапуск сервера.
# sighup - требуется перечитать файлы конфигурации.
# superuser - суперпользователь может изменить для своего сеанса.
# user - любой пользователь может изменить для своего сеанса.
```

### Порядок применения строк postgresql.conf
```shell
# Если один и тот же параметр встречается в файле несколько раз, то устанавливается значение 
# из последней считанной строки.

# Запишем в конец postgresql.conf две строки с параметром work_mem:

=> \! echo work_mem=12MB >> /usr/local/pgsql/data/postgresql.conf

=> \! echo work_mem=8MB >> /usr/local/pgsql/data/postgresql.conf

=> SELECT sourceline, name, setting, applied FROM pg_file_settings WHERE name='work_mem';
 sourceline |   name   | setting | applied 
------------+----------+---------+---------
        797 | work_mem | 12MB    | f
        798 | work_mem | 8MB     | t
(2 rows)

# По applied = f для строки с 12MB уже понятно, что это значение не будет применено.  
```

### Смена значений конфигураций
```shell
# Для параметра work_mem значение context = user. Значит, его можно менять прямо во время
# сеанса, а чтобы изменить значение во всех сеансах, достаточно перечитать файлы конфигурации.

=> SELECT pg_reload_conf();
pg_reload_conf 
----------------
 t
(1 row)

# Убедимся что work_mem получил значение от последней строки:

=> SELECT name, setting, unit, boot_val, reset_val, source, sourcefile, sourceline, pending_restart, context FROM pg_settings WHERE name = 'work_mem'\gx
-[ RECORD 1 ]---+--------------------------------------
name            | work_mem
setting         | 8192
unit            | kB
boot_val        | 4096
reset_val       | 8192
source          | configuration file
sourcefile      | /usr/local/pgsql/data/postgresql.conf
sourceline      | 798
pending_restart | f
context         | user
```

### Команда ALTER SYSTEM
```shell
# Для примера установим параметр work_mem:

=> ALTER SYSTEM SET work_mem TO '16mb';
ERROR:  invalid value for parameter "work_mem": "16mb"
HINT:  Valid units for this parameter are "B", "kB", "MB", "GB", and "TB".

# ALTER SYSTEM выполняет проверку на допустимы значения.

=> ALTER SYSTEM SET work_mem TO '16MB';
ALTER SYSTEM

# В результате выполнения команды значение 16MB записано в файл postgresql.auto.conf:

=> SELECT * FROM regexp_split_to_table(pg_read_file('postgresql.auto.conf'), '\n');
                 regexp_split_to_table                 
-------------------------------------------------------
 # Do not edit this file manually!
 # It will be overwritten by the ALTER SYSTEM command.
 work_mem = '16MB'
 
(4 rows)

# ...но не установлено:

=> SHOW work_mem;
 work_mem 
----------
 8MB
(1 row)

# Чтобы применить изменения для work_mem, перечитаем файлы конфигурации:
=> SELECT pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)

=> SELECT name, setting, unit, boot_val, reset_val, source, sourcefile, sourceline, pending_restart, context FROM pg_settings WHERE name = 'work_mem'\gx
-[ RECORD 1 ]---+-------------------------------------------
name            | work_mem
setting         | 16384
unit            | kB
boot_val        | 4096
reset_val       | 16384
source          | configuration file
sourcefile      | /usr/local/pgsql/data/postgresql.auto.conf
sourceline      | 3
pending_restart | f
context         | user
```

### Удаление строк из postgresql.auto.conf
```shell
# Для удаления строк из postgresql.auto.conf изпользуется команда ALTER SYSTEM RESET.

=> ALTER SYSTEM RESET work_mem;
ALTER SYSTEM

=> SELECT * FROM regexp_split_to_table(pg_read_file('postgresql.auto.conf'), '\n');
                 regexp_split_to_table                 
-------------------------------------------------------
 # Do not edit this file manually!
 # It will be overwritten by the ALTER SYSTEM command.
 
(3 rows)

# Еще раз перечитаем конфигруацию. Теперь восстановится значение из postgresql.conf:

=> SELECT pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)

=> SELECT name, setting, unit, boot_val, reset_val, source, sourcefile, sourceline, pending_restart, context FROM pg_settings WHERE name = 'work_mem'\gx
-[ RECORD 1 ]---+--------------------------------------
name            | work_mem
setting         | 8192
unit            | kB
boot_val        | 4096
reset_val       | 8192
source          | configuration file
sourcefile      | /usr/local/pgsql/data/postgresql.conf
sourceline      | 798
pending_restart | f
context         | user
```

### Установка параметров во время исполнения
```shell
# Для изменения параметров во время сеанса можно использовать команду SET:

=> SET work_mem TO '24MB';
SET

# Или функцию set_config:
=> SELECT set_config('work_mem', '32MB', false);
 set_config 
------------
 32MB
(1 row)

# Третий параметр функции говорит о том, нужно ли устанавливать значение только для текущей
# транзакции (true) или до конца работы сеанса (false). Это важно при работе приложения через
# пулл соединений, когда  одном сеансе могут выполняться транзакции разных пользоателей.
```

### Чтение значений параметров во время выполнения
```shell
=> SHOW work_mem;
 work_mem 
----------
 32MB
(1 row)

=> SELECT current_setting('work_mem');
 current_setting 
-----------------
 32MB
(1 row)

=> SELECT name, setting, unit FROM pg_settings WHERE name = 'work_mem';
   name   | setting | unit 
----------+---------+------
 work_mem | 32768   | kB
(1 row)
```

### Установка параметров транзационна
```shell
# Откроем транзакцию и установим новое значение work_mem:

=> RESET work_mem;
RESET

=> BEGIN;
BEGIN

=> SET work_mem TO '64MB';
SET

=> SHOW work_mem;
 work_mem 
----------
 64MB
(1 row)

# Если транзакция откатывется, то установка параметра отменяется:

=> ROLLBACK;
ROLLBACK

=> SHOW work_mem;
 work_mem 
----------
 8MB
(1 row)
```

### Пользовательские параметры
```shell
=> SELECT CASE WHEN current_setting('myapp.currency_code', true) IS NULL THEN
      set_config('myapp.currency_code', 'RUB', false)
   ELSE
      current_setting('myapp.currency_code')
   END;
   
current_setting 
-----------------
RUB
(1 row)

# В имени пользоательского параметра обязательно должна быть точка, чтобы отличать их
# от стандартных параметров.

=> SET myapp.test TO 'test';
SET

=> SHOW myapp.test;
myapp.test 
------------
test
(1 row)

# Теперь myapp.test можно использовать как глобальную переменную сеанса:

=> SELECT current_setting('myapp.test');
 current_setting 
-----------------
 test
(1 row)

# Польовательские параметры можно указать в postgresql.conf, тогда они автоматически будут 
# иннициализироватьсч во всех сеансах.
```