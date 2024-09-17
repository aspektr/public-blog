## Описание эксперимента

В данном материале мы подвергнем [[Ymatrix. Знакомство | Ymatrix]] тесту [TPC-DS](https://www.tpc.org/tpcds/)

Официальный материал гласит:

TPC-DS — это нагрузочный тест, который моделирует несколько общеприменимых аспектов системы поддержки принятия решений, включая аналитические запросы к данным и манипулирование данными. 

Этот тест обеспечивает репрезентативную оценку производительности системы поддержки принятия решений общего назначения. 

Результат теста измеряет:
* время ответа на запрос в однопользовательском режиме, 
* пропускную способность запросов в многопользовательском режиме 
* и производительность при манипулировании данными 

для заданного оборудования, операционной системы и конфигурации системы обработки данных в условиях контролируемой, сложной, многопользовательской рабочей нагрузки.

Целью тестов TPC является предоставление актуальных и объективных данных о производительности отраслевым пользователям. 

Говоря проще, тест создает серию таблиц в виде снежинки, замеряет время загрузки данных в них, а затем запускает серию аналитических запросов, фиксируя время исполнения каждого. Эта нагрузка может эмулироваться как в однопользовательском режиме, так и в многопользовательском режиме.

![[Pasted image 20240903152219.png]]


## Отношение к бенчмаркам

Организовать корректный бенчмарк очень непростая задача, еще сложнее - это сделать из него правильные выводы. А если начать сравнивать на базе теста еще и разные технологии ( а реализации теста могут варьироваться), вникая в нюансы организации бенчмарка, то легко впасть в "беличий транс". Термин пришел из маркетинга, и является ярлыком ситуации, когда человек стоит у полок магазина и не знает, что ему выбрать. 

Несколько моментов на которые стоит обратить внимание:
* TPC-DS больше свод правил и запросов для организации теста, а не конкретная его имплементация (структуры данных могут подвергаться дополнительной оптимизации)
* Исходя из п.1 очень многое в результатах теста зависит от конкретной его реализации
* Не используйте результат теста как самый весомый аргумент для принятия решения
* Не забывайте, что основная метрика теста - это не скорость работы сама по себе, а показатель «цена/производительность»

## Классы запросов

TPC-DS стремится захватить все типы запросов с которыми можно столкнуться в реальном мире.
####  Отчетные запросы

Эти запросы отражают «отчетную» природу системы принятия решений. Они включают запросы, которые выполняются периодически для ответа на известные, предопределенные вопросы о финансовом и операционном здоровье бизнеса. Хотя отчетные запросы, как правило, статичны, незначительные изменения являются обычным явлением. От одного использования отчетного запроса к другому пользователь может сместить фокус, изменив диапазон дат, географическое местоположение или название бренда.

#### Ad hoc запросы

Эти запросы отражают динамическую природу системы принятия решений, в которой импровизированные запросы создаются для ответа на немедленные и конкретные бизнес-вопросы. Основное различие между запросами ad hoc и отчетными запросами заключается в ограниченной возможности архитекторов системы прогнозировать такие запросы, а значит структуры данных могут быть неоптимальны для выполнения конкретного ad hoc запроса.

#### Итеративные OLAP запросы

OLAP запросы  позволяют исследовать и анализировать бизнес-данные для обнаружения новых и значимых взаимосвязей и тенденций. Хотя этот класс запросов похож на класс «Ad hoc», он отличается пользовательским сценарием, в котором отправляется последовательность запросов, где результат прошлого запроса может использоваться для подачи на вход следующего запроса. Такая последовательность может включать как сложные, так и простые запросы. Например, это может быть сценарий drill down (углубление в данные)

#### Запросы Data mining

Data mining - это процесс просеивания больших объемов данных для создания связей между содержанием данных. Он может предсказывать будущие тенденции и поведение, позволяя компаниям принимать проактивные решения на основе знаний. Этот класс запросов обычно состоит из объединений и больших агрегаций, которые возвращают большие наборы данных.

С примерами запросов можно познакомиться в [этом репозитории](https://github.com/Agirish/tpcds)

## Манипуляция данными

Хранилище данных может быть настолько точным и актуальным, насколько точны и актуальны операционные данные, на которых оно основано.
Миграция данных из OLTP систем в аналитические системы имеет критическое значение.
Процесс миграции, как правило, сильно отличается от  бизнеса к бизнесу, от приложения к приложению. 

Процесс обновления данных в хранилище обычно включает три отдельных шага:

**извлечение данных** - эта фаза состоит из точного извлечения данных из учетных систем и других источников бизнес данных. Тест TPC-DS считает, что хотя эти процессы важны, но процесс приобретения и настройки ETL систем отделен от приобретения и конфигурации систем принятия решений. Соответственно ETL процесс не моделируется в бенчмарке. Бенчмарк стартует с процесса генерирования плоских файлов, которые как предполагается, являются выходом процесса ETL.

**трансформация данных** - это процесс когда извлеченные данные очищаются и преобразуются в общий формат, подходящий для базы данных системы поддержки и принятия решений.

**загрузка данных** - это вставка, изменение, удаление данных в базе данных системы поддержки и принятия решений.

![[Pasted image 20240903161130.png]]

## Логическая схема данных

Продажи через фффлайн канал (магазины)
![[Pasted image 20240903161239.png]]
Возвраты через магазины
![[Pasted image 20240903161300.png]]
Продажи через канал продаж каталог
![[Pasted image 20240903161341.png]]

Возвраты по каналу продаж каталог
![[Pasted image 20240903161404.png]]

Продажи через онлайн канал продаж
![[Pasted image 20240903161545.png]]

Возвраты через онлайн канал

![[Pasted image 20240903161616.png]]

Складские запасы

![[Pasted image 20240903161735.png]]


## Фактор масштаба

TPC-DS устанавливает следующую линейку размера данных: 1Tb, 3 Tb, 10Tb, 30 Tb, 100 Tb


![[Pasted image 20240903162027.png]]

![[Pasted image 20240903162053.png]]

## Кластер для теста

Для теста был развернут кластер в Яндекс облаке на базе CentOS 7 OS Login:

| Имя VM | Платформа          | vCPU | RAM, Gb | Disk Vol, Gb | Comment      |     |
| ------ | ------------------ | ---- | ------- | ------------ | ------------ | --- |
| mdw    | Intel Cascade Lake | 8    | 32      | 100 SSD      | master node  |     |
| rdw    | Intel Cascade Lake | 8    | 32      | 100 SSD      | replica      |     |
| sdw1   | Intel Cascade Lake | 16   | 64      | 1000 HDD     | segment host |     |
| sdw2   | Intel Cascade Lake | 16   | 64      | 1000 HDD     | segment host |     |
| sdw3   | Intel Cascade Lake | 16   | 64      | 1000 HDD     | segment host |     |
| sdw4   | Intel Cascade Lake | 16   | 64      | 1000 HDD     | segment host |     |

## Кодовая база

За основу возьмем [этот репозиторий](https://github.com/RunningJon/TPC-DS?tab=readme-ov-file) в котором представлен тест для greenplum и попробуем натянуть сову на глобус. Основной приоритет будет отдаваться скорости работы, а не воспроизводимости теста, поэтому масштаб костылей обещает быть эпичным.


## Готовим тест к запуску

Проверяем статус кластера при необходимости запускаем его
```bash
 sudo su - mxadmin
```

![[Pasted image 20240903164451.png]]

```bash
mxstart -a
```

На порту `8240` висит MXUI, проверяем статус кластера через веб-морду

![[Pasted image 20240903164633.png]]

Убедимся, что мы в домашнем каталоге
```bash
pwd
```

Качаем скрипт и устанавливаем права запуска

```bash
curl https://raw.githubusercontent.com/pivotalguru/TPC-DS/master/tpcds.sh > tpcds.sh && chmod 755 tpcds.sh
```

Проверяем

```bash
ls -lah
```

Нам нужен файл `tpcds_variables.sh` , для правки параметров, которого нет в результате листинга директории.
Попробуем запустить `tpcds.sh`, чтобы этот скрипт создал нам файл параметров `tpcds_variables.sh`.

```bash
exit
sudo su -
cd /home/mxadmin/
./tpcds.sh
```

Получаем сообщение
```
There are new variables in the tpcds_variables.sh file.  Please review to ensure the values are correct and then re-run this script.`
```

В директории появился файл `tpcds_variables.sh`

```
REPO="TPC-DS"
REPO_URL="https://github.com/pivotalguru/TPC-DS"
ADMIN_USER="gpadmin"
INSTALL_DIR="/pivotalguru"
EXPLAIN_ANALYZE="false"
RANDOM_DISTRIBUTION="false"
MULTI_USER_COUNT="5"
GEN_DATA_SCALE="3000"
SINGLE_USER_ITERATIONS="1"
RUN_COMPILE_TPCDS="false"
RUN_GEN_DATA="false"
RUN_INIT="true"
RUN_DDL="true"
RUN_LOAD="true"
RUN_SQL="true"
RUN_SINGLE_USER_REPORT="true"
RUN_MULTI_USER="true"
RUN_MULTI_USER_REPORT="true"
RUN_SCORE="true"
```

Поменяем параметр `ADMIN_USER="gpadmin"`  на `ADMIN_USER="mxadmin"`
Поменяем фактор масштаба `GEN_DATA_SCALE="3000"` на `GEN_DATA_SCALE="1000"`\
## Создаем базу данных для бенчмарка

Попробуем первый запуск и посмотрим, что отвалится

```bash
./tpcds.sh > tpcds.log 2>&1 < tpcds.log
```


Судя по короткому времени, все пошло по плану т.е. без успеха. Смотрим лог

```bash
cat tpcds.log | less
```

```
psql: error: FATAL:  database "mxadmin" does not exist
```

Не удивительно, у нас нет такой базы данных, но почему именно mxadmin?
Скорее всего дело в том, что `04_load/analyze.sh` содержит следующую конструкцию:

```bash
if [ "$return_status" -eq "0" ]; then
	dbname="$PGDATABASE"
	if [ "$dbname" == "" ]; then
		dbname="$ADMIN_USER"
	fi
	if [ "$PGPORT" == "" ]; then
```

Ок. У нас нет задачи сделать правильно, лайфках для нас это создать базу `mxadmin`

```bash
exit
sudo su - mxadmin
psql postgres
```


```sql
CREATE DATABASE mxadmin;
SELECT datname from pg_database;
```

На выходе должно получиться:

```
 datname
-----------
 postgres
 template1
 template0
 matrixmgr
 mxadmin
(5 rows)

```

Выходим из командной строки базы данных
```sql
exit
```

Возвращаемся к руту

```bash
exit
sudo su -
cd /home/mxadmin/
```

На этот раз, шансов, что что-то пойдет не так стало меньше, поэтому воспользуемся более безопасной конструкцией, которая защитит нас от потери консоли

```bash
nohup ./tpcds.sh > tpcds.log 2>&1 < tpcds.log &
```

## Обнаруживаем, что тест не работает с 7ой версией GP

Получаем ошибку исполнения...

```
copy tpcds binaries to mdw:/home/mxadmin
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@       WARNING: POSSIBLE DNS SPOOFING DETECTED!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
The ECDSA host key for mdw has changed,
and the key for the corresponding IP address 10.128.0.4
is unchanged. This could either mean that
DNS SPOOFING is happening or the IP address for the host
and its host key have changed at the same time.
Offending key for IP in /home/mxadmin/.ssh/known_hosts:14
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256: .
Please contact your system administrator.
Add correct host key in /home/mxadmin/.ssh/known_hosts to get rid of this message.
Offending ED25519 key in /home/mxadmin/.ssh/known_hosts:9
ECDSA host key for mdw has changed and you have requested strict checking.
Host key verification failed.
lost connection

```

Неприятность случается на этапе копирования файла. Скорее всего используется утилита `scp`

Поиск по репозиторию подтверждает гипотезу, мы ломаемся на этом кусочке кода

```bash
#copy the compiled dsdgen program to the segment hosts
	for i in $(cat $PWD/../segment_hosts.txt); do
		echo "copy tpcds binaries to $i:$ADMIN_HOME"
		scp tools/dsdgen tools/tpcds.idx $i:$ADMIN_HOME/
	done
```

Найдем файл `segment_hosts.txt`

```bash
find / -type f -name segment_hosts.txt 2>/dev/null
```

А вот и он:

```
/pivotalguru/TPC-DS/segment_hosts.txt
```

Что внутри?

```bash
cat /pivotalguru/TPC-DS/segment_hosts.txt
```

Только одна запись:

```
mdw
```

Скорее всего проблема связана с функцией `create_hosts_file()` из `functions.sh`

```bash
create_hosts_file()
{
	get_version

	if [[ "$VERSION" == *"gpdb"* ]]; then
		psql -v ON_ERROR_STOP=1 -t -A -c "SELECT DISTINCT hostname FROM gp_segment_configuration WHERE role = 'p' AND content >= 0" -o $LOCAL_PWD/segment_hosts.txt
	else
		#must be PostgreSQL
		echo $MASTER_HOST > $LOCAL_PWD/segment_hosts.txt
	fi
}
```

Версия GP в функции  `get_version` из этого же файла определяется затейливой конструкцией:

```bash
VERSION=$(psql -v ON_ERROR_STOP=1 -t -A -c "SELECT CASE WHEN POSITION ('Greenplum Database 4.3' IN version) > 0 THEN 'gpdb_4_3' WHEN POSITION ('Greenplum Database 5' IN version) > 0 THEN 'gpdb_5' WHEN POSITION ('Greenplum Database 6' IN version) > 0 THEN 'gpdb_6' ELSE 'postgresql' END FROM version();")
```

Что выдает нам YMatrix?

```bash
exit
sudo su - mxadmin
psql mxadmin
```

```sql
select version();
```

```
 PostgreSQL 12 (MatrixDB 5.99.0-dev+enterprise) (Greenplum Database 7.0.0+dev.22114.g7dec5e6b746 build commit:7dec5e6b74
6d5d3f843b3afe633d9b4b2403d9bb) on x86_64-pc-linux-gnu, compiled by gcc (GCC) 11.2.1 20220127 (Red Hat 11.2.1-9), 64-bit
```

Скрипт староват для нас. У нас 7ая версия GP, скрипт может корректно работать только с 6ой версией. И так как автор не заложил 7ую версию в свою работу, скрипт принимает нас за postgres standalone. И далее работает только с мастер нодой `mdw`. Нам же нужно раскидать данные по кластеру.

Проверим, что возвращает конструкция:

```sql
SELECT DISTINCT hostname FROM gp_segment_configuration WHERE role = 'p' AND content >= 0
```
Вот ее ответ:

```
 sdw1.ru-central1.internal
 sdw2.ru-central1.internal
 sdw3.ru-central1.internal
 sdw4.ru-central1.internal
```

Этот ответ нас устраивает.
Возможное лекарство может выглядеть следующим образом:

```sql
SELECT 
	CASE WHEN POSITION ('Greenplum Database 4.3' IN version) > 0 THEN 'gpdb_4_3' WHEN POSITION ('Greenplum Database 5' IN version) > 0 THEN 'gpdb_5' WHEN POSITION ('Greenplum Database 7' IN version) > 0 THEN 'gpdb_6' ELSE 'postgresql' END FROM version();
```

Мы подменим `Greenplum Database 6` на `Greenplum Database 7`
Запуск измененной конструкции возвращает желаемый результат:

```
  case
--------
 gpdb_6
(1 row)
```

Классическая же конструкция из скрипта для YMatrix возвращает:

```

  case
------------
 postgresql
(1 row)

```

## Решаем проблему с ssh ключами для root

Вернемся к этому моменту позже, разберемся с `scp`. Вернемся к `root`

```bash
exit
exit
sudo su -
cd /home/mxadmin/
```

Нам нужно имитировать команду

```bash
scp tools/dsdgen tools/tpcds.idx $i:$ADMIN_HOME/
```

Создадим нагрузочный файл:

```bash
echo "123" > test.txt
```

И попробуем пробросить файл

```bash
scp test.txt sdw1.ru-central1.internal:/home/mxadmin/
```

Ответ консоли:

```
The authenticity of host 'sdw1.ru-central1.internal (10.128.0.6)' can't be established.
ECDSA key fingerprint is SHA256:.
ECDSA key fingerprint is MD5:.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'sdw1.ru-central1.internal,10.128.0.6' (ECDSA) to the list of known hosts.
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
lost connection

```

Однако, под учеткой `mxadmin` все работает:

```bash 
sudo su - mxadmin
cd /home/mxadmin/
scp test.txt sdw1.ru-central1.internal:/home/mxadmin/
```

Очень грубо и некультурно перебросим ключи `root`:

```bash
exit
sudo su -
cd $HOME
cp -r /home/mxadmin/.ssh/* /root/.ssh/

```

Делаем тест:

```bash
cd /home/mxadmin/
scp test.txt sdw1.ru-central1.internal:/home/mxadmin/
```

Получаем:

```
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).

```

В целом это неудивительно, так как мы лезем по умолчанию, как `root` на машину `sdw1`. Следующая конструкция легко нас пропустит на машину (ключи то у нас от `mxadmin`!):

```bash 
ssh mxadmin@sdw1.ru-central1.internal
```

Скажем еще раз спасибо за то, что скрипт работает из под `root` и продолжим.
На машине `sdw1` нам  нужен `root`, через учетку `mxadmin` фокус не получится.

Пробросим закрытый ключ с моей `win` машины на `mdw` c помощью утилиты `pscp`, чтобы можно было зайти как `someone` на машину `sdw1` и далее использовать возможности `sudo`

```bash
 pscp -i C:\Users\user\.ssh\key.ppk C:\Users\user\.ssh\id_rsa someone@ipaddress:/home/someone/.ssh
```

Далее:

```bash
ssh someone@sdw1
sudo su -
cp -r /home/mxadmin/.ssh/* /root/.ssh/
logout
logout
```

Все это, повторю, крайне грубо, неаккуратно и небезопасно...
Переключаемся на root на машине `mdw` и проверяем доступ через `ssh`

```bash
 ssh sdw1
```

SSH работает

Выйдем и вернемся к тесту scp
```bash
logout
cd /home/mxadmin
scp test.txt sdw1.ru-central1.internal:/home/mxadmin/

```

На этот раз все прошло успешно.

Повторим операцию для трех оставшихся хостов: `sdw2` `sdw3` `sdw4`

## Отключаем скрипт от репозитория

Возвращаемся к проблеме, где скрипт принимает нас за `postgres` и отказывается работать с нашими хостами. Дело было в функции `create_hosts_file()` из файла `functions.sh`

```bash
create_hosts_file()
{
	get_version

	if [[ "$VERSION" == *"gpdb"* ]]; then
		psql -v ON_ERROR_STOP=1 -t -A -c "SELECT DISTINCT hostname FROM gp_segment_configuration WHERE role = 'p' AND content >= 0" -o $LOCAL_PWD/segment_hosts.txt
	else
		#must be PostgreSQL
		echo $MASTER_HOST > $LOCAL_PWD/segment_hosts.txt
	fi
}
```

Если мы обратим внимание на файл `tpcds.sh` еще раз, то его устройство сводится к последовательному запуску функций:

```
check_user
check_variables
yum_installs
repo_init
script_check
echo_variables
```

выглядит так, что он каждый раз инициализирует репу. Такое поведение для нас нежелательно, так как нам потребуется внести изменения в код (костыли) и иметь гарантию, что очередной запуск скрипта их не перезапишет.

Закомментируем "лишние" строки

```bash
check_user
check_variables
#yum_installs
#repo_init
#script_check
echo_variables
```

## Переводим скрипт на работу с 7ой версией GP

В файле `tpcds_variables.sh` у нас лежит установочный путь: `INSTALL_DIR="/pivotalguru"`
Там находится кодовая база, для которой мы подготовили фейслифтинг.

Нужную нам правку внесем в файл `functions.sh`
Нас интересуют строки:

```bash
get_version()
{
	#need to call source_bashrc first
	VERSION=$(psql -v ON_ERROR_STOP=1 -t -A -c "SELECT CASE WHEN POSITION ('Greenplum Database 4.3' IN version) > 0 THEN 'gpdb_4_3' WHEN POSITION ('Greenplum Database 5' IN version) > 0 THEN 'gpdb_5' WHEN POSITION ('Greenplum Database 6' IN version) > 0 THEN 'gpdb_6' ELSE 'postgresql' END FROM version();") 
```


Открываем vi

```bash
vi /pivotalguru/TPC-DS/functions.sh
```

и меняем `Greenplum Database 6` на `Greenplum Database 7`

Потестим...

```bash
./tpcds.sh > tpcds.log 2>&1 < tpcds.log
```

## Подселяем greenplum_path.sh
\
Получаем новый квест в логе

```
does not contain greenplum_path.sh
```

Имеем дело все с тем же злополучным `functions.sh`

```bash
if [ "$count" -eq "0" ]; then
		get_version
		if [[ "$VERSION" == *"gpdb"* ]]; then
			echo "$startup_file does not contain greenplum_path.sh"
			echo "Please update your $startup_file for $ADMIN_USER and try again."
			exit 1
		fi
```

Найдем `greenplum_path.sh`

```bash
find / -type f -name greenplum_path.sh 2>/dev/null
```

и посмотрим его содержимое

```bash
cat /opt/ymatrix/matrixdb5/greenplum_path.sh
```

А вот и оно

```bash
if test -n "${ZSH_VERSION:-}"; then
    # zsh
    SCRIPT_PATH="${(%):-%x}"
elif test -n "${BASH_VERSION:-}"; then
    # bash
    SCRIPT_PATH="${BASH_SOURCE[0]}"
else
    # Unknown shell, hope below works.
    # Tested with dash
    result=$(lsof -p $$ -Fn | tail --lines=1 | xargs --max-args=2 | cut --delimiter=' ' --fields=2)
    SCRIPT_PATH=${result#n}
fi

if test -z "$SCRIPT_PATH"; then
    echo "The shell cannot be identified. \$GPHOME may not be set correctly." >&2
fi
SCRIPT_DIR="$(cd "$(dirname "${SCRIPT_PATH}")" >/dev/null 2>&1 && pwd)"

if [ ! -L "${SCRIPT_DIR}" ]; then
    GPHOME=${SCRIPT_DIR}
else
    GPHOME=$(readlink "${SCRIPT_DIR}")
fi
PYTHONPATH="${GPHOME}/lib/python"
PATH="${GPHOME}/bin:${PATH}"
LD_LIBRARY_PATH="${GPHOME}/lib${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"

if [ -e "${GPHOME}/etc/openssl.cnf" ]; then
        OPENSSL_CONF="${GPHOME}/etc/openssl.cnf"
fi

export GPHOME
export PATH
export PYTHONPATH
export LD_LIBRARY_PATH
export OPENSSL_CONF
export OPENBLAS_NUM_THREADS=1

# source matrixdb default configuration
source /etc/default/matrixdb5

```

Попробуем особо не разбираясь в логике подсорсить этот файл в `functions.sh`

```bash
source_bashrc()
{
        if [ -f ~/.bashrc ]; then
                # don't fail if an error is happening in the admin's profile
                source ~/.bashrc || true
        fi
        count=$(grep -v "^#" ~/.bashrc | grep "greenplum_path" | wc -l)
        if [ "$count" -eq "0" ]; then
                get_version
                if [[ "$VERSION" == *"gpdb"* ]]; then
                        echo "$startup_file does not contain greenplum_path.sh"
                        echo "Please update your $startup_file for $ADMIN_USER and try again."
                        source /opt/ymatrix/matrixdb5/greenplum_path.sh
                        #exit 1
                fi
        fi
}

```

Я закомментировал `exit 1` и добавил `source /opt/ymatrix/matrixdb5/greenplum_path.sh`
Стартуем тест

```bash
nohup ./tpcds.sh > tpcds.log 2>&1 < tpcds.log &
```

## Раскидываем данные для загрузки в таблицы по кластеру

Мониторинг на всех сегментах показывает одну и туже картину

![[Pasted image 20240903223103.png]]

Ситуация на мастер ноде

![[Pasted image 20240903223140.png]]

Реплика
![[Pasted image 20240903223242.png]]

На то, чтобы сгенерировать 1Тб данных и раскидать их по кластеру потребовался 1 час.

## Перезагрузка конфига кластера

Далее скрипт споткнулся на конструкции в файле `02_init/rollout.sh`

```bash
if [ "$update_config" -eq "1" ]; then
		echo "update cluster because of config changes"
		gpstop -u
	fi
```

У YMatrix нет команды `gpstop -u`, но есть `mxstop -u`
Внесем изменения в файл, чтобы получилось :

```bash
if [ "$update_config" -eq "1" ]; then
		echo "update cluster because of config changes"
		mxstop -u
	fi
```

Следующая задача: у нас уже раскиданы файлы с данными по кластеру и этот этап не хочется повторять, необходимо вмешаться в поток исполнения скрипта.

За это отвечает небольшой цикл в файле `rollout.sh`

```bash
for i in $(ls -d $PWD/0*); do
	echo "$i/rollout.sh"
	$i/rollout.sh $GEN_DATA_SCALE $EXPLAIN_ANALYZE $RANDOM_DISTRIBUTION $MULTI_USER_COUNT $SINGLE_USER_ITERATIONS
done
```

Проверим гипотезу

```bash
cat tpcds.log | grep rollout.sh
```

Гипотеза подтверждается

```
/pivotalguru/TPC-DS/00_compile_tpcds/rollout.sh
/pivotalguru/TPC-DS/01_gen_data/rollout.sh
/pivotalguru/TPC-DS/02_init/rollout.sh

```

Мы сломались на втором шаге. C него и продолжим. 
Сначала "потренируемся на кошках":

```bash
for i in $(ls -d  /pivotalguru/TPC-DS/0*); do
	if [ "$i" == "/pivotalguru/TPC-DS/00_compile_tpcds" ] || [ "$i" == "/pivotalguru/TPC-DS/01_gen_data" ]; then
	    continue   #Go to next iteration of I in the loop and skip statements
	fi
	echo "$i/rollout.sh"
done
```

При запуске получаем

```
/pivotalguru/TPC-DS/02_init/rollout.sh
/pivotalguru/TPC-DS/03_ddl/rollout.sh
/pivotalguru/TPC-DS/04_load/rollout.sh
/pivotalguru/TPC-DS/05_sql/rollout.sh
/pivotalguru/TPC-DS/06_single_user_reports/rollout.sh
/pivotalguru/TPC-DS/07_multi_user/rollout.sh
/pivotalguru/TPC-DS/08_multi_user_reports/rollout.sh
/pivotalguru/TPC-DS/09_score/rollout.sh

```

Боевой код примет вид

```bash
for i in $(ls -d $PWD/0*); do
	if [ "$i" == "/pivotalguru/TPC-DS/00_compile_tpcds" ] || [ "$i" == "/pivotalguru/TPC-DS/01_gen_data" ]; then
		    continue   #Go to next iteration of I in the loop and skip statements
	fi
	echo "$i/rollout.sh"
	$i/rollout.sh $GEN_DATA_SCALE $EXPLAIN_ANALYZE $RANDOM_DISTRIBUTION $MULTI_USER_COUNT $SINGLE_USER_ITERATIONS
```

Вносим изменения и запускаемся

```bash
nohup ./tpcds.sh > tpcds.log 2>&1 < tpcds.log &
```

## Решаем проблему с запуском gpfdist

Следующая проблема

```
Unable to start gpfdist on port 4001
```

`root` не знает путь к `gpfdist`, проверим путь из под учетной записи `mxadmin`

```bash
which gpfdist
```

```
/opt/ymatrix/matrixdb5/bin/gpfdist
```

Мы можем дать `root`/`mxadmin` такую возможность, добавив строку в файл `.bash_profile`

```bash
PATH=$PATH:/opt/ymatrix/matrixdb5/bin
```


>Но эту операцию надо проделать на каждом сегменте!!!

Сделаем менее изящно, но очень надежно. Впишем полный путь в файл `04_load/start_gpfdist.sh` 

Вместо 

```bash
gpfdist -p $GPFDIST_PORT -d $GEN_DATA_PATH > gpfdist.$GPFDIST_PORT.log 2>&1 < gpfdist.$GPFDIST_PORT.log &
```

у нас будет

```bash
/opt/ymatrix/matrixdb5/bin/gpfdist -p $GPFDIST_PORT -d $GEN_DATA_PATH > gpfdist.$GPFDIST_PORT.log 2>&1 < gpfdist.$GPFDIST_PORT.log &
```

Запустим тест
```bash
nohup ./tpcds.sh > tpcds.log 2>&1 < tpcds.log &
```


## Проблемы с интерконнектом

и нас ожидает новая проблема:

```
psql -v ON_ERROR_STOP=1 -f /pivotalguru/TPC-DS/04_load/001.gpdb.time_dim.sql | grep INSERT | awk -F ' ' '{print $3}'
psql -v ON_ERROR_STOP=1 -f /pivotalguru/TPC-DS/04_load/002.gpdb.ship_mode.sql | grep INSERT | awk -F ' ' '{print $3}'
psql -v ON_ERROR_STOP=1 -f /pivotalguru/TPC-DS/04_load/003.gpdb.reason.sql | grep INSERT | awk -F ' ' '{print $3}'
psql -v ON_ERROR_STOP=1 -f /pivotalguru/TPC-DS/04_load/004.gpdb.income_band.sql | grep INSERT | awk -F ' ' '{print $3}'
psql -v ON_ERROR_STOP=1 -f /pivotalguru/TPC-DS/04_load/005.gpdb.item.sql | grep INSERT | awk -F ' ' '{print $3}'
psql -v ON_ERROR_STOP=1 -f /pivotalguru/TPC-DS/04_load/006.gpdb.date_dim.sql | grep INSERT | awk -F ' ' '{print $3}'
psql -v ON_ERROR_STOP=1 -f /pivotalguru/TPC-DS/04_load/007.gpdb.customer_demographics.sql | grep INSERT | awk -F ' ' '{print $3}'
psql:/pivotalguru/TPC-DS/04_load/007.gpdb.customer_demographics.sql:1: WARNING:  interconnect may encountered a network error, please check your network  (seg10 slice1 10.128.0.8:6002 pid=12567)
DETAIL:  Failing to send packet (seq 26) to 10.128.0.7:27154 (pid 9925 cid 4) after 100 retries.
psql:/pivotalguru/TPC-DS/04_load/007.gpdb.customer_demographics.sql:1: WARNING:  terminating connection because of crash of another server process
DETAIL:  The postmaster has commanded this server process to roll back the current transaction and exit, because another server process exited abnormally and possibly corrupted shared memory.
HINT:  In a moment you should be able to reconnect to the database and repeat your command.
psql:/pivotalguru/TPC-DS/04_load/007.gpdb.customer_demographics.sql:1: server closed the connection unexpectedly
        This probably means the server terminated abnormally
        before or while processing the request.
psql:/pivotalguru/TPC-DS/04_load/007.gpdb.customer_demographics.sql:1: fatal: connection to server was lost
```

По всей видимости, начались проблемы с интерконнектом. Мы ведь не ожидали, что Яндекс Облако предоставит по умолчанию гигабитную сеть между нодами.


Продолжение следует...


