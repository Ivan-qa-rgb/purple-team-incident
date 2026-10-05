# Индикаторы компрометации (IOCs)

Приведены обезличенные значения (RFC 5737, RFC 2606).

## Сетевые IOCs

| Тип | Значение | Комментарий |
|-----|----------|-------------|
| IP | 203.0.113.10 | Аномальный ASN |
| Домен | example-cdn.invalid | Служебный ресурс |
| JA3 | `<hash>` | Клиент не из корпоративного парка |
| URL | `https://example-cdn.invalid/upload` | Точка выгрузки |

## Хостовые IOCs

| Тип | Значение |
|-----|----------|
| Путь | `/var/www/sharepoint/cache/.update` |
| Процесс | `w3wp.exe → cmd.exe → powershell.exe` |
| Учётная запись | `svc-update-01` |
| Scheduled Task | `\MicrosoftEdgeUpdateTaskMACHINE` |
| Registry | `HKLM\Software\Microsoft\Windows\CurrentVersion\Run\Updater` |

## Идентичность

- Новые OAuth-приложения в арендаторе
- Входы с анонимных прокси и незнакомых ASN
- Impossible Travel
- Массовые обновления refresh-токенов
- Отключение MFA для конкретных учётных записей

## Рекомендации по использованию

- Загрузить IOC в SIEM и EDR как watchlist.
- Не блокировать по одному признаку — использовать корреляцию.
- Регулярно обновлять список.
