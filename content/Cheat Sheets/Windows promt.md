---
title: windows promt
draft: false
tags:
---
| Команда                                                        | Описание                                                                             |
| -------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| `xfreerdp /v:<target IP address> /u:htb-student /p:<password>` | RDP на `target` машину                                                               |
| `Get-WmiObject -Class win32_OperatingSystem`                   | Информация об операционной системе                                                   |
| `dir c:\ /a`                                                   | Листинг каталога `С:\`                                                               |
| `tree <directory>`                                             | Графическое отображение структуры каталогов                                          |
| `tree c:\ /f \| more`                                          | Постраничное отображение структуры каталогов                                         |
| `icacls <directory>`                                           | Отображение прав на каталог                                                          |
| `icacls c:\users /grant user:f`                                | Назначение полных прав пользователю на каталог                                       |
| `icacls c:\users /remove user`                                 | Отзыв всех прав у пользователя на каталог                                            |
| `Get-Service`                                                  | Просмотр запущенных сервисов                                                         |
| `help <command>`                                               | Вызвать подсказку по комманде                                                        |
| `get-alias`                                                    | Псевдонимы комманд                                                                   |
| `New-Alias -Name "Show-Files" Get-ChildItem`                   | Установка нового псевдонима                                                          |
| `Get-Module \| select Name,ExportedCommands \| fl`             | Просмотр импортированных в `PowerShell` модулей                                      |
| `Get-ExecutionPolicy -List`                                    | Просмотр политики выполнения команд `PowerShell`'ом                                  |
| `Set-ExecutionPolicy Bypass -Scope Process`                    | Установка разрешающий политики на выполнение комманд в текущей сессии `PowerShell`'а |
| `wmic os list brief`                                           | Получить информацию об операционной системе с помощью `wmic`                         |
| `Invoke-WmiMethod`                                             | Вызвать метод `WMI` объекта                                                          |
| `whoami /user`                                                 | Просмотр SID  пользователя                                                           |
| `reg query <key>`                                              | Просмотр информации о ключе регистра                                                 |
| `Get-MpComputerStatus`                                         |                                                                                      |
| `sconfig`                                                      |                                                                                      |
