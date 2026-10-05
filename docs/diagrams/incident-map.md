# Карта инцидента

## Цепочка атаки

```mermaid
flowchart TD
    A[🌐 Интернет] -->|сканирование| B[🖥️ SharePoint<br/>публичный доступ]
    B -->|T1190: эксплуатация| C[💻 RCE<br/>w3wp.exe]
    C -->|T1059: PowerShell| D[📁 Сбор секретов<br/>web.config, токены]
    D -->|T1098: OAuth| E[☁️ OAuth-приложение<br/>Entra ID]
    D -->|T1078: сервисная учётка| F[🔑 Active Directory]
    F -->|T1087: разведка| G[🔍 Discovery<br/>LDAP, net group]
    G -->|T1021: SMB| H[📂 Файловые серверы<br/>C$, ADMIN$]
    H -->|T1005: collection| I[📤 Выгрузка данных<br/>T1567]
    I -->|T1486: подготовка| J[🔒 Шифрование<br/>не началось]
    J -->|🚨 SOC| K[✅ Обнаружение<br/>MTTD 12ч]
    K -->|🛡️ IR| L[✅ Локализация<br/>MTTC 3ч]
```

## Тактики ATT&CK

```mermaid
graph LR
    A[Initial Access<br/>T1190] --> B[Execution<br/>T1059]
    B --> C[Persistence<br/>T1136, T1098]
    C --> D[Privilege Escalation<br/>T1078]
    D --> E[Defense Evasion<br/>T1562]
    E --> F[Credential Access<br/>T1552, T1003]
    F --> G[Discovery<br/>T1087]
    G --> H[Lateral Movement<br/>T1021]
    H --> I[Collection<br/>T1005]
    I --> J[Exfiltration<br/>T1567]
    J --> K[Impact<br/>T1486]
```

## Временная шкала

```mermaid
timeline
    title Хронология инцидента
    section 03 января
        23:41 : Сканирование сервиса
    section 04 января
        00:05 : Эксплуатация уязвимости
        00:18 : Первый дочерний процесс
        01:48 : Создание учётной записи
        02:15 : OAuth-приложение
        08:10 : Первая выгрузка данных
        12:00 : Обнаружение SOC
        12:20 : Локализация
    section 05-07 января
        : Восстановление
        : Ротация секретов
        : Post-mortem
```

## Сеть до инцидента

```mermaid
flowchart LR
    subgraph Интернет
        A[Атакующий]
    end
    
    subgraph Периметр
        B[SharePoint<br/>публичный доступ]
    end
    
    subgraph Корпоративная сеть
        C[Active Directory]
        D[Файловые серверы]
        E[Бэкап-сервер]
    end
    
    A -->|443/tcp| B
    B --> C
    B --> D
    C --> D
    D --> E
```

## Сеть после улучшений

```mermaid
flowchart LR
    subgraph Интернет
        A[Атакующий]
    end
    
    subgraph Периметр
        B[WAF]
        C[SharePoint<br/>VPN доступ]
    end
    
    subgraph Корпоративная сеть
        D[Active Directory<br/>MFA]
        E[Файловые серверы<br/>сегментация]
        F[Бэкап-сервер<br/>отдельный домен]
    end
    
    A -->|блокировка| B
    B --> C
    C -->|JIT/JEA| D
    D --> E
    E -.->|нет доступа| F
```
