---
title: windows promt
draft: false
tags:
---
| Команда                                                                                                       | Описание                                                                                     |
| ------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| `xfreerdp /v:<target IP address> /u:htb-student /p:<password>`                                                | RDP на `target` машину                                                                       |
| `Get-WmiObject -Class win32_OperatingSystem`                                                                  | Информация об операционной системе                                                           |
| `dir c:\ /a`                                                                                                  | Листинг каталога `С:\`                                                                       |
| `tree <directory>`                                                                                            | Графическое отображение структуры каталогов                                                  |
| `tree c:\ /f \| more`                                                                                         | Постраничное отображение структуры каталогов                                                 |
| `icacls <directory>`                                                                                          | Отображение прав на каталог                                                                  |
| `icacls c:\users /grant user:f`                                                                               | Назначение полных прав пользователю на каталог                                               |
| `icacls c:\users /remove user`                                                                                | Отзыв всех прав у пользователя на каталог                                                    |
| `Get-Service`                                                                                                 | Просмотр запущенных сервисов                                                                 |
| `help <command>`                                                                                              | Вызвать подсказку по комманде                                                                |
| `get-alias`                                                                                                   | Псевдонимы комманд                                                                           |
| `New-Alias -Name "Show-Files" Get-ChildItem`                                                                  | Установка нового псевдонима                                                                  |
| `Get-Module \| select Name,ExportedCommands \| fl`                                                            | Просмотр импортированных в `PowerShell` модулей                                              |
| `Get-ExecutionPolicy -List`                                                                                   | Просмотр политики выполнения команд `PowerShell`'ом                                          |
| `Set-ExecutionPolicy Bypass -Scope Process`                                                                   | Установка разрешающий политики на выполнение комманд в текущей сессии `PowerShell`'а         |
| `wmic os list brief`                                                                                          | Получить информацию об операционной системе с помощью `wmic`                                 |
| `Invoke-WmiMethod`                                                                                            | Вызвать метод `WMI` объекта                                                                  |
| `whoami /user`                                                                                                | Просмотр SID  пользователя                                                                   |
| `reg query <key>`                                                                                             | Просмотр информации о ключе регистра                                                         |
| `Get-MpComputerStatus`                                                                                        | Просмотр работающих защит компьютера                                                         |
| `sconfig`                                                                                                     | Утилита для настройки Windows Server Core                                                    |
| `gpedit.msc`                                                                                                  | Запуск редактора локальных политик безопасностей                                             |
| `Get-MpComputerStatus`                                                                                        | Просмотр включенных защит компьютера                                                         |
| `services.msc`                                                                                                | Запуск Service Control Manager (SCM)                                                         |
| ````powershell-session<br>Get-Service \| ? {$_.Status -eq "Running"} \| select -First 2 \|fl<br>````          | Вывод списка запущенных сервисов в командную строку                                          |
| ```cmd-session<br>\\live.sysinternals.com\tools\procdump.exe -accepteula<br>```                               | Запуск [онлайн-утилит](https://docs.microsoft.com/en-us/sysinternals) без скачивания на диск |
| ```cmd-session<br>sc qc wuauserv<br>```                                                                       | Опрос сервиса с помощью утилиты sc                                                           |
| ```cmd-session<br>sc sdshow wuauserv<br>```                                                                   | Листинг прав сервиса                                                                         |
| ```powershell-session<br>Get-ACL -Path HKLM:\System\CurrentControlSet\Services\wuauserv \| Format-List<br>``` | Листинг прав сервиса в `PowerShell`                                                          |
| ```powershell-session<br>.\PowerView.ps1;Get-LocalGroup \|fl<br>```                                           | просмотр групп                                                                               |
| ```powershell-session<br>.\PowerView.ps1;Get-LocalUser \|fl<br>```                                            | просмотр пользователей                                                                       |
| `mmc`                                                                                                         | Запуск Microsoft Management Console (MMC)                                                    |
| `eventvwr`                                                                                                    | Команда для запуска Microsoft Event Viewer                                                   |
