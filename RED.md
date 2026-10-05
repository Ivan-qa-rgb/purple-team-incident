# Purple Team — Атака (Red Team)

**Класс документа:** Purple Team — Attack Scenarios  
**Классификация:** Internal / учебный  
**Версия:** 1.0  
**Дата:** 2026-01-08  

## 1.1. Профиль угрозы

**Класс:** ransomware / extortion с предварительной выгрузкой данных  
**Мотив:** финансовый  
**Уровень:** целевая атака, mixed skill (initial access и ransomware — разные команды)  
**Основной вектор:** эксплуатация уязвимости в интернет-доступном корпоративном сервисе

## 1.2. Цели атаки

| Приоритет | Цель |
|---|---|
| 1 | Получить удалённое выполнение кода на периметре |
| 2 | Собрать учётные данные и токены |
| 3 | Закрепиться в инфраструктуре |
| 4 | Получить доступ к домену |
| 5 | Обесточить защиту |
| 6 | Выгрузить данные |
| 7 | Подготовить шифрование |

## 1.3. Тактики и техники (ATT&CK-маппинг)

### Фаза 1. Initial Access

| Техника | ID | Описание |
|---|---|---|
| Exploit Public-Facing Application | T1190 | Эксплуатация уязвимости в публичном сервисе |
| Valid Accounts: Cloud Accounts | T1078.004 | Использование украденных облачных учётных данных |
| Trusted Relationship | T1199 | Использование доверенного контрагента |

**Artifacts:**

- аномальные entries в WAF/IIS-логах;
- нестандартные User-Agent;
- запросы к admin-эндпоинтам;
- повышенный уровень ошибок HTTP;
- сканирование параметров приложения.

### Фаза 2. Execution

| Техника | ID | Описание |
|---|---|---|
| Command and Scripting Interpreter: PowerShell | T1059.001 | Запуск PS от веб-процесса |
| Command and Scripting Interpreter: Windows Command Shell | T1059.003 | Запуск cmd от w3wp.exe |
| Native API | T1106 | Использование нативных API ОС |

**Artifacts:**

- цепочка процессов `w3wp.exe → cmd.exe → powershell.exe`;
- PowerShell с `-EncodedCommand`, `-NoProfile`, `-WindowStyle Hidden`;
- загрузки через `IWR`, `DownloadString`, `Invoke-WebRequest`;
- обращения к `AMSI` bypass (детектируется);
- командные строки, содержащие base64 payload.

### Фаза 3. Persistence

| Техника | ID | Описание |
|---|---|---|
| Create Account: Local Account | T1136.001 | Локальная учётная запись |
| Create Account: Domain Account | T1136.002 | Доменная учётная запись |
| Additional Cloud Credentials | T1098.001 | OAuth-приложение |
| Scheduled Task/Job | T1053.005 | Планировщик задач |
| Server Software Component: Web Shell | T1505.003 | Веб-шелл на сервере |

**Artifacts:**

- новые локальные/доменные учётные записи вне окна change-окна;
- новые OAuth-приложения в Entra ID;
- новые scheduled tasks с непрозрачными именами;
- новые файлы в корне web-root;
- изменения `web.config` и `.htaccess`.

### Фаза 4. Privilege Escalation

| Техника | ID | Описание |
|---|---|---|
| Valid Accounts: Domain Accounts | T1078.002 | Использование сервисной учётки |
| Exploitation for Privilege Escalation | T1068 | Локальный privesc |
| Steal or Forge Kerberos Tickets | T1558 | Kerberoasting, AS-REP Roasting |

**Artifacts:**

- входы с сервисных учётных записей вне baseline;
- аномальные запросы TGS (4769);
- обращения к `lsass.exe`;
- дампы памяти (`comsvcs.dll`, `procdump`, `Task Manager`).

### Фаза 5. Defense Evasion

| Техника | ID | Описание |
|---|---|---|
| Impair Defenses: Disable or Modify Tools | T1562.001 | Остановка EDR |
| Impair Defenses: Disable Windows Event Logging | T1562.002 | Отключение аудита |
| Obfuscated Files or Information | T1027 | Обфускация |
| Indicator Removal: Clear Windows Event Logs | T1070.001 | Очистка журналов |
| Masquerading: Match Legitimate Name | T1036.005 | Маскировка утилит |

**Artifacts:**

- `Set-MpPreference -DisableRealtimeMonitoring $true`;
- остановка служб EDR-агента;
- изменение политик аудита через `auditpol`;
- `wevtutil cl` и очистка журналов;
- запуск инструментов под легитимными именами (`svchost.exe` вне System32).

### Фаза 6. Credential Access

| Техника | ID | Описание |
|---|---|---|
| Unsecured Credentials: Credentials in Files | T1552.001 | Чтение конфигов |
| Unsecured Credentials: Env Variables | T1552.007 | Переменные окружения |
| OS Credential Dumping: LSASS Memory | T1003.001 | Дамп LSASS |
| Credentials from Password Stores | T1555 | Хранилища паролей |

**Artifacts:**

- чтение `web.config`, `appsettings.json`, `.env`;
- доступ к `HKLM\SECURITY\Policy\Secrets`;
- доступ к `lsass.exe` от необычных процессов;
- массовые обращения к файловым шарам;
- создание `.dmp`-файлов.

### Фаза 7. Discovery

| Техника | ID | Описание |
|---|---|---|
| Account Discovery: Domain Account | T1087.002 | Перечисление доменных учёток |
| Network Share Discovery | T1135 | Перечисление шар |
| Remote System Discovery | T1018 | Перечисление хостов |
| Domain Trust Discovery | T1482 | Обнаружение доверий |

**Artifacts:**

- LDAP-запросы к DC;
- `net group "Domain Admins" /domain`;
- `nltest /domain_trusts`;
- `net view`, `net share`;
- адресные DNS-запросы к внутренним подсетям.

### Фаза 8. Lateral Movement

| Техника | ID | Описание |
|---|---|---|
| Remote Services: SMB/Windows Admin Shares | T1021.002 | SMB-шары |
| Remote Services: RDP | T1021.001 | RDP |
| Remote Services: WinRM | T1021.006 | WinRM |
| Use Alternate Authentication Material | T1550 | Pass-the-Hash |

**Artifacts:**

- входы 4624 типа 3 и 10;
- обращения к `C$`, `ADMIN$`, `IPC$`;
- удалённое создание сервисов (`sc.exe \\host create`);
- WMI-вызовы (`wmic /node:...`);
- RDP-сессии вне baseline.

### Фаза 9. Collection

| Техника | ID | Описание |
|---|---|---|
| Data from Local System | T1005 | Локальные файлы |
| Data from Network Shared Drive | T1039 | Сетевые диски |
| Data from Information Repositories | T1213 | SharePoint, Confluence |
| Data from Cloud Storage | T1530 | Облачные хранилища |

**Artifacts:**

- массовое чтение документов одним пользователем;
- скачивание через API облачных сервисов;
- обращения к почтовым ящикам;
- выгрузки из BI-систем.

### Фаза 10. Exfiltration

| Техника | ID | Описание |
|---|---|---|
| Exfiltration Over Web Service | T1567.002 | Выгрузка через веб |
| Exfiltration Over C2 Channel | T1041 | Выгрузка через C2 |
| Transfer Data to Cloud Account | T1537 | Выгрузка в облако |

**Artifacts:**

- большие POST-запросы на внешние ресурсы;
- передачи через `rclone`, `megacmd`, `azcopy`;
- аномалии NetFlow;
- использование сервисов, не входящих в allowlist.

### Фаза 11. Impact

| Техника | ID | Описание |
|---|---|---|
| Data Encrypted for Impact | T1486 | Шифрование |
| Inhibit System Recovery | T1490 | Удаление бэкапов и теневых копий |
| Service Stop | T1489 | Остановка сервисов |

**Artifacts:**

- `vssadmin delete shadows /all`;
- `wbadmin delete catalog`;
- `bcdedit /set {default} recoveryenabled No`;
- изменение MBR;
- остановка служб резервного копирования;
- появление новых расширений файлов.

## 1.4. Сценарии для red team

### Сценарий A. Периметр → внутренняя сеть

```text
Внешний сервис → RCE → сбор секретов → AD-разведка → SMB → файловые серверы
```

### Сценарий B. Идентичность → облако

```text
Украденный токен → OAuth-app → доступ к почте и файлам → выгрузка данных
```

### Сценарий C. Защита → отключение

```text
Локальные права → попытка остановить EDR → отключение аудита → очистка журналов
```

## 1.5. Правила для red team

- Не выполнять действия, нарушающие закон и договоры.
- Не выходить за согласованный scope.
- Не публиковать PoC и рабочие эксплойты.
- Немедленно останавливаться при риске повреждения данных.
- Всё взаимодействие с защитой — только в контролируемых окнах.

## 1.6. Ограничения эмуляции

- Не используются реальные эксплойты против прода.
- Не применяются реальные персональные данные.
- Все действия только в изолированной лаборатории.
- Запрещены действия, нарушающие закон и договоры.
- Немедленная остановка при риске повреждения данных.

## 1.7. Milestones для отчётности

| Этап | Метрика |
|---|---|
| Вход в периметр | Время до RCE |
| Закрепление | Время до первого persistence |
| Достижение AD | Время до компрометации DA |
| Доступ к бэкапам | Время до бэкап-сервера |
| Выгрузка | Объём данных без детекта |
| Impact | Полное или частичное шифрование |
