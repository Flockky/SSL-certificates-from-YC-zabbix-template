# Zabbix Templates: SSL Certificates Monitoring Collection

Набор шаблонов для мониторинга сроков действия SSL-сертификатов из различных источников в Zabbix 7.4+.

## Обзор шаблонов

| Шаблон | Источник данных | Метод получения | Триггеры (дней) |
| :--- | :--- | :--- | :--- |
| **PKI2 SSL Сертификаты** | Внутренняя PKI | HTTP Agent (JSON URL) | < 30 (Avg), < 7 (High) |
| **Lets Encrypt SSL** | Let's Encrypt (Certbot) | System.run (Local JSON) | < 30 (Avg), < 14 (High) |
| **International SSL** | Reg.ru API | HTTP Agent (POST) | < 30 (Avg), < 7 (High) |
| **HARICA SSL** | HARICA CA | External Check (Script) | < 30 (Avg), < 7 (High) |

---

## 1. PKI2 SSL Сертификаты (`PKI2 SSL certs`)

Мониторинг сертификатов внутреннего центра сертификации.

### Настройка
1. Импортируйте шаблон.
2. На хосте задайте макрос `{$PKI2.JSON.URL}` — ссылка на endpoint, отдающий JSON.

### Ожидаемый формат JSON
Источник должен возвращать массив объектов:

```json
[
  {
    "Dead_Line": "7",
    "Short_hash": "a1b2",
    "Ticket": " \"123456\"",
    "Expiration_Time": "06/04/2026 05:30:49",
    "Hash": "a1 b2 c3 d4 ...",
    "Template_name": null,
    "Cert_name": "app-server-01.example.com",
    "DNS_name": "app-server-01.example.com lb.example.com"
  },
  {
    "Dead_Line": "0",
    "Short_hash": "e5f6",
    "Ticket": " \"789012\"",
    "Expiration_Time": "05/28/2026 11:18:54",
    "Hash": "e5 f6 g7 h8 ...",
    "Template_name": null,
    "Cert_name": "db-master.example.com",
    "DNS_name": "db-master.example.com db-replica.example.com"
  }
]
```

---

## 2. Lets Encrypt SSL Сертификаты (`Lets Encrypt SSL certs`)

Мониторинг локальных сертификатов Let's Encrypt.

### Настройка
1. Разместите скрипт генерации JSON на хосте.
2. Задайте макрос `{$LE.CERTS.SCRIPT_PATH}` — путь к файлу с JSON.
3. Убедитесь, что `EnableRemoteCommands=1` в агенте или используйте UserParameter.

### Ожидаемый формат JSON
Файл по пути скрипта должен содержать:

```json
[
  {
    "name": "api.example.com",
    "serial_number": "abc123def456...",
    "key_type": "RSA",
    "domains": ["api.example.com"],
    "expiration_date": "2026-08-17 12:26:26+00:00",
    "validity": "81 days",
    "certificate_path": "/etc/letsencrypt/live/api.example.com/fullchain.pem",
    "private_key_path": "/etc/letsencrypt/live/api.example.com/privkey.pem",
    "expiration_date_epoch": 1786958786,
    "expiration_date_epoch_utc": 1786969586,
    "expiration_date_iso": "2026-08-17T12:26:26+00:00"
  },
  {
    "name": "portal.example.com",
    "serial_number": "xyz789uvw012...",
    "key_type": "RSA",
    "domains": ["portal.example.com"],
    "expiration_date": "2026-05-29 09:27:25+00:00",
    "validity": "1 day",
    "certificate_path": "/etc/letsencrypt/live/portal.example.com/fullchain.pem",
    "private_key_path": "/etc/letsencrypt/live/portal.example.com/privkey.pem",
    "expiration_date_epoch": 1780036045,
    "expiration_date_epoch_utc": 1780046845,
    "expiration_date_iso": "2026-05-29T09:27:25+00:00"
  }
]
```

---

## 3. Международные SSL Сертификаты (`International SSL certs`)

Мониторинг через API регистратора (Reg.ru).

### Настройка
1. Импортируйте шаблон.
2. Задайте макросы:
   * `{$REGRU.USERNAME}` — Логин API.
   * `{$REGRU.PASSWORD}` — Пароль API (тип Secret Text).

### Ожидаемый формат JSON
Шаблон парсит поле `answer.services` из ответа API. Пример массива услуг:

```json
[
  {
    "subtype": "gs_domainssl_dns",
    "service_id": 12345678,
    "uplink_service_id": 0,
    "expiration_date": "2026-06-07",
    "servtype": "srv_ssl_certificate",
    "creation_date": "2025-05-06",
    "state": "A",
    "dname": "api.example.com",
    "expiration_days": 9
  },
  {
    "subtype": "",
    "service_id": 87654321,
    "uplink_service_id": 0,
    "expiration_date": "2027-03-05",
    "servtype": "domain",
    "creation_date": "2022-03-05",
    "state": "A",
    "dname": "myshop.ru",
    "expiration_days": 280
  }
]
```

---

## 4. HARICA SSL Сертификаты (`HARICA SSL certs`)

Мониторинг сертификатов HARICA через внешний скрипт.

### Настройка
1. Поместите скрипт `HaricaSSL.sh` в `/usr/lib/zabbix/externalscripts/`.
2. `chmod +x` и права `zabbix:zabbix`.
3. Макрос `{$HARICA.SCRIPT_PARAMS}` оставьте пустым, если параметры не нужны.

### Ожидаемый формат JSON
Скрипт должен выводить в stdout:

```json
[
  {
    "domain": "mail.example.com",
    "notBefore": 1759845940,
    "notAfter": 1791381940,
    "daysLeft": "132"
  },
  {
    "domain": "auth.example.com",
    "notBefore": 1759846323,
    "notAfter": 1791382323,
    "daysLeft": "132"
  },
  {
    "domain": "legacy.example.com",
    "notBefore": null,
    "notAfter": null,
    "daysLeft": "null"
  }
]
```

---

## Требования и Troubleshooting

*   **Zabbix:** 7.4+
*   **SELinux:** Если используются External Scripts или `system.run`, убедитесь, что политики не блокируют выполнение.
    ```bash
    setsebool -P httpd_can_network_connect 1
    ```
*   **Отладка:** Если данные не приходят, проверьте элемент данных "Get data" в **Latest Data**. Используйте вкладку **Preprocessing** для просмотра исходного JSON и ошибок парсинга.

## Лицензия

MIT
```
