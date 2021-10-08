# Использование psql
 
## Назначение
- Терминальный клиент для работы с PostgreSQL
- Поставляется вместе с СУБД
- Используется администраторами и разработчиками для интерактивной работы и выполнения скриптов

Для работы с СУБД PostgreSQL существуют различные сторонние
инструменты, рассмотрение которых не входит в рамки курса.
В курсе мы будем использовать терминальный клиент psql:
- `psql` — это единственный клиент, поставляемый вместе с СУБД. 
- Навыки работы c `psql` пригодятся разработчикам и администраторам
   БД вне зависимости от того, с каким инструментом они будут работать
   дальше.
   https://postgrespro.ru/docs/postgresql/10/app-psql.html

## Подключение

- Запуск `$ psql -d база -U роль -h узел -p порт`
- Новое подключение в psql  `=> \c[onnect] база роль узел порт`
- Информация о текущем подключении `=> \conninfo`

При запуске `psql` нужно указать параметры подключения.
К обязательным параметрам подключения относятся: имя базы данных,
имя пользователя, имя сервера, номер порта. Если эти параметры не
указаны, `psql` попробует подключиться, используя значения по
умолчанию:
- база — совпадает с именем пользователя;
- роль — совпадает с именем пользователя ОС;
- узел — локальное соединение;
- порт — обычно 5432.

Значения по умолчанию позволяют пользователю postgres
подключаться к PosgtreSQL без указания параметров.
Если требуется выполнить новое подключение не выходя из `psql`, то
нужно выполнить команду `\connect`.
Команда `\conninfo` выдает информацию о текущем подключении.

Дополнительная информация о возможностях настройки подключения:
https://postgrespro.ru/docs/postgresql/10/libpq-envars.html
https://postgrespro.ru/docs/postgresql/10/libpq-pgservice.html
https://postgrespro.ru/docs/postgresql/10/libpq-pgpass.html

## Получение справки

* В командной строке ОС
  * `$ psql --help`
  * `$ man psql`
* В psql
  * => `\?` список команд psql
  * => `\? variables` переменные psql
  * => `\h[elp]` список команд SQL
  * => `\h команда` синтаксис команды SQL
  * => `\q` выход

Справочную информацию по psql можно получить не только в
документации, но и прямо в системе.

`psql` с ключом `--help` выдает справку по запуску. А если в системе
была установлена документация, то справочное руководство можно
получить командой `man psql`.

`psql` умеет выполнять команды SQL и свои собственные команды.
Внутри `psql` есть возможность получить список и краткое описание
команд `psql`. Все команды psql начинаются с обратной косой черты (\).
Команда `\help` выдает список команд SQL, которые поддерживает
сервер, а также синтаксис конкретной команды SQL.


И еще одна команда, которую полезно знать, хоть она и не имеет
отношения к справке, это `\q` — выход из psql.

## Файлы

* Скрипты при запуске `psql`
  * общий системный файл `psqlrc`
  * пользовательские файлы `~/.psqlrc`
  

* История команд
  * пользовательские файлы `~/.psql_history`
  * поиск по истории с помощью `readline`
  * количество хранимых команд можно изменить


При запуске `psql` выполняются команды, записанные в двух файлах —
общесистемном и пользовательском (если они есть, конечно).

Общий системный файл называется `psqlrc` и располагается в каталоге
`/usr/local/pgsql/etc` при обычной сборке из исходных кодов.

Расположение этого каталога можно узнать командой
`pg_config --sysconfdir`.

Пользовательский файл находится в домашнем каталоге пользователя
ОС и называется `.psqlrc`. Его расположение можно изменить, задав
переменную окружения `PSQLRC`.

В эти файлы можно записать команды, настраивающие `psql` —
например, изменить приглашение, включить вывод времени
выполнения команд и т. п.

История команд сохраняется в файле `.psql_history` в домашнем
каталоге пользователя. 

Расположение этого файла можно изменить,
задав переменную окружения `PSQL_HISTORY` или переменную `psql
HISTFILE`. 

По умолчанию хранится 500 последних команд; это число
можно изменить переменной `psql HISTSIZE`.

Пролистывать историю команд можно стрелками вверх и вниз, искать с
помощью `Ctrl+R` — доступен весь набор команд, предлагаемых `readline`.

## Практика

### Запускаем `psql`
```shell
postgres$ psql
psql (14.0)
Type "help" for help.
=> 

# Проверим текущее соединение:
=> \conninfo
You are connected to database "postgres" as user "postgres" via socket in "/tmp" at port "5432".

# Команда \c[onnect] выполняет новое подключение, не покида psql.
```
### Формат вывода
```shell
# Формат с выравниванием используется по умолчанию:

=> SELECT schemaname, tablename, tableowner FROM pg_tables LIMIT 5;
 schemaname |       tablename       | tableowner 
------------+-----------------------+------------
 pg_catalog | pg_statistic          | postgres
 pg_catalog | pg_type               | postgres
 pg_catalog | pg_foreign_table      | postgres
 pg_catalog | pg_authid             | postgres
 pg_catalog | pg_statistic_ext_data | postgres
(5 rows)

# Ширина столбцов выровнена по значениям. Также выводится строка заголовка и итоговаря
# строка.
```
Команды `psql` для переключения режимов выравнивания
* `\a` - переключатель режима: c выравниванием/без выравнивания.
* `\t` - переключатель отображения строки заголовка и итоговой строки.
```shell
=> \t\a
Tuples only is on.
Output format is unaligned.

=> SELECT schemaname, tablename, tableowner FROM pg_tables LIMIT 5;
pg_catalog|pg_statistic|postgres
pg_catalog|pg_type|postgres
pg_catalog|pg_foreign_table|postgres
pg_catalog|pg_authid|postgres
pg_catalog|pg_statistic_ext_data|postgres

=> \pset fieldsep ' '
Field separator is " ".

=> SELECT schemaname, tablename, tableowner FROM pg_tables LIMIT 5;
pg_catalog pg_statistic postgres
pg_catalog pg_type postgres
pg_catalog pg_foreign_table postgres
pg_catalog pg_authid postgres
pg_catalog pg_statistic_ext_data postgres

# Расширенный режим удобен, когда нужно вывести много столбцов для одной
# или нескольких записей:

=> \x
Expanded display is on.

=> SELECT * FROM pg_tables WHERE tablename = 'pg_class';
-[ RECORD 1 ]-----------
schemaname  | pg_catalog
tablename   | pg_class
tableowner  | postgres
tablespace  | 
hasindexes  | t
hasrules    | f
hastriggers | f
rowsecurity | f

# Все возможности форматирования запросов в команде `\pset`.
```
### Взаимодействие с ОС
```shell
=> \!pwd
/var/lib/postgresql

=> \! uptime
08:13:33 up  2:28,  1 user,  load average: 1,94, 2,59, 2,77
```
### Переменные ОС
```shell
\setenv Name User
\! echo $Name
User
```
### Вывод в файл
```shell
# Можно записать вывод команды в файл с помощью `\o[ut];`

=> \o dbal_log

=> SELECT schemaname, tablename, tableowner, FROM pg_tables LIMIT 5;

=> \! cat dbal_log
 schemaname |       tablename       | tableowner 
------------+-----------------------+------------
 pg_catalog | pg_statistic          | postgres
 pg_catalog | pg_type               | postgres
 pg_catalog | pg_foreign_table      | postgres
 pg_catalog | pg_authid             | postgres
 pg_catalog | pg_statistic_ext_data | postgres
(5 rows)
```