# Отчёт по лабораторной работе «Поиск аномалий HTTP-трафика по PCAP»

**ФИО: Эрцеговац Данила Желькович**

**Группа: P4150**

**Дата: 17.06.2026**

**Версия стенда: 1**

В рамках работы также вёл конспект по [теории](https://github.com/danilaercegovac/Network/blob/main/3rd_Individual_Lab/theoretical_konspekt.md).
---

## 1. Стенд

* Команды, которыми поднимали стенд:
  ```
  wsl -l -v
  wsl --status # Ubuntu был, дополнительно не устанавливал его
  docker --version
  docker compose version # Docker также оказался
  wsl -d Ubuntu-20.04 # Подключил Ubuntu
  cd "/mnt/c/Users/Данила Эрцеговац/OneDrive/Документы/network-anomaly-lab" # Встал в свою папку стенда
  ls # Убедился что всё на месте
  cp .env.example .env # Скопировал шаблон для рабочего файла настроек среды
  make up # Попробовал поднять стенд. Не получилось - ошибка
  make --version # Увидел, что докер не доступен для Ubuntu
  (Зашёл в Dpcoker Desktop -> Settings -> Resources -> WSL Integration. Включил Enable integration with additional distros: Ubuntu-20.04)
  docker --version 
  docker compose version # Убедился, что докер стал доступен
  make up (docker compose up -d --build) # Поднял стенд
  ```
* `docker compose ps` :
  ```
  NAME          IMAGE                COMMAND                  SERVICE             CREATED         STATUS                   PORTS
  lab_backend   lab/backend:latest   "uvicorn app.main:ap…"   backend             6 minutes ago   Up 6 minutes (healthy)   8000/tcp
  lab_nginx     lab/nginx:latest     "/docker-entrypoint.…"   nginx               6 minutes ago   Up 5 minutes (healthy)   127.0.0.1:8080->80/tcp
  lab_sensor    lab/sensor:latest    "sh -lc 'echo 'senso…"   sensor              6 minutes ago   Up 5 minutes            
  lab_traffic   lab/traffic:latest   "sh -lc 'echo 'traff…"   traffic-generator   6 minutes ago   Up 5 minutes
  ```
* SHA-256 PCAP-файла: `c491fc1a4c2167385fa7752e017497ea9a0653e72a07dc5e372536aa29da9f43`
* Длительность захвата: 90 сек.

---

## 2. Профиль нормального трафика

Нормальный трафик представлен последовательностью пользовательских действий: регистрация, авторизация, просмотр каталога товаров, поиск, просмотр профиля и загрузка файлов.

Основными HTTP-методами являются GET и POST. Наиболее часто встречаются обращения к URI `/api/auth/login`, `/api/auth/register`, `/api/products`, `/api/search`, `/api/profile` и `/api/upload`.

Преобладают успешные ответы сервера с кодом 200. В качестве User-Agent используются строки, характерные для обычных браузеров Chrome и Firefox под Windows и Linux.

Запросы поступают равномерно без резких всплесков интенсивности, что соответствует поведению обычного пользователя.

---

## 3. Таблица найденных аномалий

| № | Тип аномалии          | Время (start / end)             | Источник (IP) | Назначение (IP) | Endpoint (METHOD URI) | Сетевой признак                                      | Доказательство                                           | Вывод                                                 |
| - | --------------------- | ------------------------------- | ------------- | --------------- | --------------------- | ---------------------------------------------------- | -------------------------------------------------------- | ----------------------------------------------------- |
| 1 | Suspicious User Agent | 1781756904.736 / 1781756904.957 | 172.28.0.4    | 172.28.0.3      | GET /api/products     | Использование нетипичных User-Agent                  | curl/8.4.0, python-requests/2.31.0, sqlmap, пустой UA    | Вероятное автоматизированное сканирование             |
| 2 | Directory Traversal   | 1781756905.457 / 1781756905.670 | 172.28.0.4    | 172.28.0.3      | GET /api/files?...    | Наличие ../, %2e, %2f, обращений к системным файлам  | ../../../../etc/passwd, etc/shadow, etc/hosts            | Попытка обхода каталогов и доступа к системным файлам |
| 3 | SQL Injection Probes  | 1781756906.170 / 1781756906.438 | 172.28.0.4    | 172.28.0.3      | GET /api/search?q=... | SQL-подобные конструкции в параметрах                | OR 1=1, UNION SELECT, admin'--                           | Попытка проверки приложения на SQL-инъекции           |
| 4 | Credential Stuffing   | 1781756906.938 / 1781756909.749 | 172.28.0.4    | 172.28.0.3      | POST /api/auth/login  | Большое число запросов авторизации за короткое время | 120+ попыток входа, детектор зафиксировал 181 POST login | Попытка подбора учётных данных                        |
| 5 | Large Upload          | 1781756910.249 / 1781756910.282 | 172.28.0.4    | 172.28.0.3      | POST /api/upload      | Аномально большой объём данных в запросе             | Content-Length ≈ 204985 байт (~200 KB)                   | Подозрительная загрузка крупного объекта              |
| 6 | HTTP Flood            | 1781756911.923 / 1781756931.965 | 172.28.0.4    | 172.28.0.3      | GET /api/products     | Резкое увеличение частоты запросов                   | ~285 запросов, пик детектора 43 RPS                      | Признаки HTTP Flood / DoS активности                  |

---

## 4. Детектор

Что добавлено в `detector_template.py`:
* `detect_suspicious_user_agents`: поиск запросов с User-Agent `curl`, `python-requests`, `sqlmap` и пустым User-Agent.
* `detect_directory_traversal`: поиск признаков обхода каталогов (`..`, `%2e`, `%2f`, `etc/passwd`, `etc/shadow`) в URI.
* `detect_sqli_probes`: выявление SQL-подобных конструкций в запросах (`OR 1=1`, `UNION`, `--`, `;`, `'`).
* `detect_credential_stuffing`: обнаружение большого количества запросов `POST /api/auth/login`, характерных для подбора паролей.
* `detect_large_upload`: выявление POST-запросов с аномально большим размером тела (`Content-Length > 100000` байт).
* `detect_http_flood`: обнаружение всплесков частоты HTTP-запросов путём подсчёта количества запросов в секунду и поиска пиковых значений.


Фрагменты `reports/detections_student.json`:

```json
{
  "schema": "network-anomaly-lab/detections/student/v1",
  "csv": "/reports/http_flows.csv",
  "rows": 1618,
  "findings": [
    {
      "type": "suspicious_user_agent",
      "severity": "low",
      "description": "подозрительные user-agent",
      "samples": [
        {
          "ts": 1781756904.737032,
          "ip_src": "172.28.0.4",
          "uri": "/api/products",
          "ua": "curl/8.4.0"
        },
        {
          "ts": 1781756904.796683,
          "ip_src": "172.28.0.4",
          "uri": "/api/products",
          "ua": "python-requests/2.31.0"
        },
        {
          "ts": 1781756904.849997,
          "ip_src": "172.28.0.4",
          "uri": "/api/products",
          "ua": ""
        },
        {
          "ts": 1781756904.903221,
          "ip_src": "172.28.0.4",
          "uri": "/api/products",
          "ua": "sqlmap/1.7.11#stable (https://sqlmap.org)"
        },
        {
          "ts": 1781756905.457596,
          "ip_src": "172.28.0.4",
          "uri": "/api/files?name=%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd",
          "ua": "curl/8.4.0"
        },
        {
          "ts": 1781756905.511428,
          "ip_src": "172.28.0.4",
          "uri": "/api/files?name=%2F..%252f..%252fetc%2Fpasswd",
          "ua": "curl/8.4.0"
        },
        {
          "ts": 1781756905.564691,
          "ip_src": "172.28.0.4",
          "uri": "/api/files?name=%2F%252e%252e%2F%252e%252e%2Fetc%2Fshadow",
          "ua": "curl/8.4.0"
        },
        {
          "ts": 1781756905.617725,
          "ip_src": "172.28.0.4",
          "uri": "/api/files?name=%2Fstatic%2F..%2F..%2Fetc%2Fhosts",
          "ua": "curl/8.4.0"
        },
        {
          "ts": 1781756906.171184,
          "ip_src": "172.28.0.4",
          "uri": "/api/search?q=%27%20OR%201%3D1%20--",
          "ua": "sqlmap/1.7.11#stable (https://sqlmap.org)"
        },
        {
          "ts": 1781756906.225115,
          "ip_src": "172.28.0.4",
          "uri": "/api/search?q=%22%20OR%20%22a%22%3D%22a",
          "ua": "sqlmap/1.7.11#stable (https://sqlmap.org)"
        }
      ]
    },
    {
      "type": "directory_traversal",
      "severity": "medium",
      "description": "path traversal",
      "samples": [
        {
          "ts": 1781756905.457596,
          "ip_src": "172.28.0.4",
          "uri": "/api/files?name=%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd"
        },
        {
          "ts": 1781756905.511428,
          "ip_src": "172.28.0.4",
          "uri": "/api/files?name=%2F..%252f..%252fetc%2Fpasswd"
        },
        {
          "ts": 1781756905.564691,
          "ip_src": "172.28.0.4",
          "uri": "/api/files?name=%2F%252e%252e%2F%252e%252e%2Fetc%2Fshadow"
        },
        {
          "ts": 1781756905.617725,
          "ip_src": "172.28.0.4",
          "uri": "/api/files?name=%2Fstatic%2F..%2F..%2Fetc%2Fhosts"
        }
      ]
    },
    {
      "type": "sql_injection_probes",
      "severity": "medium",
      "description": "SQL injection probes",
      "samples": [
        {
          "ts": 1781756906.171184,
          "ip_src": "172.28.0.4",
          "uri": "/api/search?q=%27%20OR%201%3D1%20--"
        },
        {
          "ts": 1781756906.331975,
          "ip_src": "172.28.0.4",
          "uri": "/api/search?q=UNION%20SELECT%20username%2C%20password%20FROM%20users"
        },
        {
          "ts": 1781756906.385361,
          "ip_src": "172.28.0.4",
          "uri": "/api/search?q=admin%27--"
        }
      ]
    },
    {
      "type": "credential_stuffing",
      "severity": "high",
      "description": "массовые попытки логина",
      "samples": [
        {
          "count": 181
        }
      ]
    },
    {
      "type": "large_upload",
      "severity": "medium",
      "description": "крупная загрузка",
      "samples": [
        {
          "ts": 1781756910.256898,
          "ip_src": "172.28.0.4",
          "size": 204985
        }
      ]
    },
    {
      "type": "http_flood",
      "severity": "high",
      "description": "всплеск HTTP запросов",
      "samples": [
        {
          "peak_rps": 43
        }
      ]
    }
  ]
}
```

---

## 5. Выводы

* Наиболее устойчивыми сетевыми признаками оказались характерные строки User-Agent, признаки directory traversal в URI, SQL-инъекционные конструкции (`UNION`, `OR 1=1`, `--`) и резкие всплески частоты запросов. Эти признаки хорошо выделяются на фоне нормального трафика и редко встречаются при обычной работе приложения.

* Наибольший риск ложных срабатываний дают правила для крупных загрузок и высокой интенсивности запросов. Большой размер файла может быть результатом легитимной загрузки пользователем, а высокий RPS может возникать при автоматизированном тестировании, мониторинге или работе нескольких клиентов одновременно.

* Значительную часть обнаруженных атак можно было бы предотвратить средствами защиты приложения:

  * rate limiting для ограничения частоты запросов и защиты от HTTP Flood;
  * ограничение количества попыток входа и временная блокировка учётной записи для защиты от Credential Stuffing;
  * обязательная авторизация и контроль доступа к административным ресурсам;
  * фильтрация и валидация пользовательского ввода для предотвращения SQL Injection и Directory Traversal;
  * ограничения на размер загружаемых файлов для защиты от подозрительных загрузок.
