Деплой будем производить в Яндекс Облаке на обычные виртуальные машины.

Сайзинг кластера следующий:

| VM   | Platform       | vCPU | RAM, Gb | Disk Vol, Gb | Comment      | internal ip |
| ---- | -------------- | ---- | ------- | ------------ | ------------ | ----------- |
| mdw  | Intel Ice Lake | 8    | 32      | 100 SSD      | master node  | 10.128.0.4  |
| rdw  | Intel Ice Lake | 8    | 32      | 100 SSD      | replica      | 10.128.0.5  |
| sdw1 | Intel Ice Lake | 16   | 64      | 1000 HDD     | segment host | 10.128.0.6  |
| sdw2 | Intel Ice Lake | 16   | 64      | 1000 HDD     | segment host | 10.128.0.7  |
| sdw3 | Intel Ice Lake | 16   | 64      | 1000 HDD     | segment host | 10.128.0.8  |
| sdw4 | Intel Ice Lake | 16   | 64      | 1000 HDD     | segment host | 10.128.0.9  |
##  Базовая проверка VM

| **Шаг** | **Команда**             | **Комментарий**                    |
| ------- | ----------------------- | ---------------------------------- |
| 1       | free -h                 | Проверка объема RAM                |
| 2       | df -h                   | Проверка доступного места на диске |
| 3       | lscpu                   | Количество CPU                     |
| 4       | cat /etc/system-release | Версия операционной системы        |
| 5       | uname -a                | Информация о ядре                  |
| 6       | tail -11 /proc/cpuinfo  | Информация о CPU                   |

## Подготовка к установке

Копируем RPM пакет на машину. Для `windows` используем утилиту `pscp`

```
pscp -i C:\Users\user\.ssh\secret_key.ppk C:\upload\matrixdb5-5.99.0+enterprise~dev-1.el7.x86_64.rpm user@ipaddress:/home/user/
```

Подключаемся по ssh. Устанавливаем зависимости

```bash
sudo sed -i s/mirror.centos.org/vault.centos.org/g /etc/yum.repos.d/*.repo
sudo sed -i s/^#.*baseurl=http/baseurl=http/g /etc/yum.repos.d/*.repo
sudo sed -i s/^mirrorlist=http/#mirrorlist=http/g /etc/yum.repos.d/*.repo
sudo yum update
sudo yum install centos-release-scl
sudo yum install rh-python36
scl enable rh-python36 bash
```

## Изменяем конфигурацию ВМ

```bash
sudo systemctl stop firewalld.service
sudo systemctl disable firewalld.service
sudo sed s/^SELINUX=.*$/SELINUX=disabled/ -i /etc/selinux/config
sudo setenforce 0
```

Правим `/etc/hosts`

```bash
sudo vi /etc/hosts
```

Он должен иметь такое содержание:

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

10.128.0.4      mdw
10.128.0.5      rdw
10.128.0.6      sdw1
10.128.0.7      sdw2
10.128.0.8      sdw3
10.128.0.9      sdw4

```

Далее создадим образ диска в Яндекс Облаке, чтобы не повторять эти действия. И последующие машины развернем из этого образа.

Следующие действия необходимо повторить на каждой машине

```bash
sudo hostnamectl set-hostname mdw #rdw sdw1 sdw2 sdw3 sdw4 для других машин
sudo yum install matrixdb5-5.99.0+enterprise~dev-1.el7.x86_64.rpm
```

Сконфигурировать порты после установки можно на мастере, внеся изменения в файл `/etc/matrixdb5/defaults.conf`

## Деплой базы данных

Используем MXGUI, доступ к которому можно получить на порту `8240`
`http://external_ip:8240`

Пароль для доступа к MXGUI можно получить командой

```bash
sudo more /etc/matrixdb5/auth.conf
```

Выбираем multi-node deployment

![[Pasted image 20240904134521.png]]

Нажимаем add

![[Pasted image 20240904134559.png]]

Вводим:

![[Pasted image 20240904134651.png]]

Конфигурируем кластер следующим образом

![[Pasted image 20240904134925.png]]


Superuser password поставим 1 для удобства


Выбираем галочку

![[Pasted image 20240904135026.png]]

![[Pasted image 20240904135103.png]]


## После инсталляции

YMatrix поддерживает удаленное подключение по умолчанию. Если во время установки не был отмечен флажок «Разрешить удаленное подключение к базе данных», вручную измените файл $MASTER_DATA_DIRECTORY/pg_hba.conf, добавив следующую строку, чтобы разрешить доступ с любого IP-адреса. Все пользователи базы данных будут аутентифицироваться по паролю, и вы можете ограничить диапазон IP-адресов или имя базы данных, чтобы минимизировать риск безопасности:

```
host  all       all   0.0.0.0/0  md5
```

После внесения этих изменений вам необходимо переключиться на пользователя mxadmin и выполнить следующую команду, чтобы база данных перезагрузила файл конфигурации pg_hba.conf:

```
su - mxadmin

mxstop -u
```

| **Команда** | **Комментарий**                                                                                                                                       |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| mxstop -a   | Остановка кластера. (В этом режиме завершения работы база данных зависает, если есть  сеансы)                                                         |
| mxstop -af  | Быстрая остановка кластера                                                                                                                            |
| mxstop -arf | Перезапуск кластера. Ожидание завершения текущего выполняемого оператора SQL (В этом режиме завершения работы база данных зависает, если есть сеансы) |
| mxstate -s  | Статус кластера                                                                                                                                       |
