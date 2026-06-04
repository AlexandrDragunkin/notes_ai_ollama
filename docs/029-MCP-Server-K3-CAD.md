---
title: MCP Server для CAD K3-Мебель
description: Архитектура и реализация MCP сервера для работы с закрытыми библиотеками CAD-приложения К3-Мебель через Model Context Protocol
author: Александр Драгункин
date: 2025-06-04
---

# MCP Server для CAD K3-Мебель

## Проблема

Модуль `k3` — это встроенное C-расширение CAD-приложения **К3-Мебель** (разработка НВЦ ГеоС). Он доступен **только внутри процесса CAD**. При попытке `import k3` в обычном интерпретаторе Python возникает `ImportError`.

Все модули из директории `Proto/` используют `import k3`:

| Модуль | Назначение |
|--------|-----------|
| `mProfile` | Работа с профилями |
| `based_build` | Базовые строительные блоки (панели, ящики, шкафы) |
| `engine` | Движок мебельных объектов |
| `mCarcase` | Каркасы |
| `mPanel_utilites` | Утилиты панелей |
| `core_k` | Ядро системы |
| `database` | Работа с БД |

MCP сервер по стандарту работает как **отдельный процесс** через Stdio transport. Он не может напрямую импортировать `k3`.

## Архитектура решения

### Принципиальная схема

**Основной вариант (рекомендуемый) — CAD как TCP-сервер:**

```
┌─────────────────────────────────────────────────────────────────────┐
│                      Внешняя среда (Python 3.x)                      │
│                                                                      │
│  ┌──────────────┐         ┌──────────────────────────────────┐      │
│  │  SourceCraft │◄───────►│         MCP Server               │      │
│  │  Code Assist.│ JSON-RPC│    (k3_mcp_server.py)            │      │
│  │              │  stdio  │    TCP-клиент                    │      │
│  └──────────────┘         └──────────┬───────────────────────┘      │
│                                      │                              │
│                                      │ TCP-соединение               │
│                                      │ localhost:9000               │
│                                      ▼                              │
│                          ┌──────────────────────────┐               │
│                          │   MCP Server (TCP client) │               │
│                          │   socket.connect(9000)    │               │
│                          └──────────────────────────┘               │
└─────────────────────────────────────────────────────────────────────┘
                                      │
                                      │ TCP socket (localhost:9000)
                                      ▼
┌─────────────────────────────────────────────────────────────────────┐
│              CAD K3 (Mebel.exe -m:k3_agent_tcp.py)                   │
│                    Python 3.7 (32-bit)                               │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │              K3 Agent (TCP-сервер)                            │   │
│  │              socket.bind(('localhost', 9000))                 │   │
│  │              socket.listen()                                  │   │
│  │                                                               │   │
│  │  ┌─────────────┐  ┌──────────────┐  ┌────────────────────┐   │   │
│  │  │ import k3   │  │ import       │  │ import             │   │   │
│  │  │             │  │ mProfile     │  │ based_build        │   │   │
│  │  └─────────────┘  └──────────────┘  └────────────────────┘   │   │
│  │                                                               │   │
│  │  Полный доступ ко всем API CAD K3                             │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

**Запуск CAD как TCP-сервера:**

```bash
# Прямой запуск из командной строки:
"C:\ARL8\Bin\Mebel.exe" -m:C:\REPO\ARLINE\k3-mcp-server\k3_agent_tcp.py

# Или через .mac файл:
"C:\ARL8\Bin\Mebel.exe" -m:startapp.mac
```

Где `startapp.mac` содержит:
```python
<?python
import k3_agent_tcp
?>
LoadOrder  last ;
```

**Альтернативные варианты IPC** (для специальных случаев) описаны ниже.

### Компоненты

#### 1. MCP Server (`k3_mcp_server.py`)

Внешний Python-скрипт, работающий через Stdio transport (стандартный протокол MCP). Принимает JSON-RPC запросы от AI-ассистента, преобразует их в команды для CAD, отправляет через TCP-сокет, возвращает результат.

**Совместимость с AI-ассистентами кода:**

MCP (Model Context Protocol) — это открытый стандарт, который поддерживается различными AI-ассистентами:

| Ассистент | Поддержка MCP | Статус |
|-----------|--------------|--------|
| **SourceCraft Code Assistant** | ✅ Полная | Текущий ассистент |
| Roo Code | ✅ Полная | Альтернатива |
| Claude Desktop | ✅ Полная | Через клиентское приложение |
| Cursor | 🚧 В разработке | Планируется поддержка MCP |
| VS Code Copilot | ❌ | Не поддерживает MCP |

> **Важно:** Данный MCP сервер реализован по стандартному протоколу MCP (Model Context Protocol) и будет работать с любым ассистентом, поддерживающим этот протокол. На данный момент это **SourceCraft Code Assistant** и **Roo Code**.

**Предоставляемые tools:**

| Tool | Описание | Пример использования |
|------|----------|---------------------|
| `k3_execute` | Выполнить произвольный Python-код внутри CAD | Получить материал профиля, создать объект, изменить атрибут |
| `k3_get_material` | Получить ID материала объекта по UnitPos | Узнать, из какого материала сделан профиль/панель |
| `k3_get_order_info` | Получить информацию о текущем заказе | Исполнитель, заказчик, доп.информация |
| `k3_list_objects` | Получить список объектов в сцене по фильтру | Найти все профили с определённым атрибутом |
| `k3_get_attribute` | Получить значение атрибута объекта | Прочитать `UnitPos`, `FurnType`, `elemname` |

#### 2. IPC — TCP (основной, рекомендуемый)

**Самый простой и надёжный способ.** CAD K3 запускается из командной строки с Python-скриптом, который открывает TCP-сокет и слушает входящие запросы. MCP сервер подключается к этому сокету как TCP-клиент.

```
MCP Server (TCP-клиент)              CAD K3 (TCP-сервер)
   │                                      │
   │  socket.connect(('localhost', 9000)) │
   │─────────────────────────────────────►│
   │                                      │
   │  {"code": "import k3; ..."}         │
   │─────────────────────────────────────►│
   │                                      │── exec(code)
   │                                      │── захват stdout/stderr
   │  {"stdout": "...", "success": true}  │
   │◄─────────────────────────────────────│
   │                                      │
   │  "stop_server"                       │
   │─────────────────────────────────────►│── k3.quit()
```

**Плюсы:**
- **Минимум зависимостей** — только стандартный `socket` (встроен в Python)
- **Простота** — CAD запускается одной командой, не нужно ничего настраивать
- **Скорость** — TCP-сокет быстрее HTTP и RabbitMQ
- **Управление** — можно отправить команду `stop_server` для корректного завершения CAD

**Минусы:** Только точка-точка (один MCP сервер на один CAD)

#### 3. Альтернативные варианты IPC

Для специальных случаев доступны альтернативные варианты IPC-моста:

##### Вариант A — HTTP (для одного CAD, альтернатива TCP)

```
MCP Server                          K3 Agent (в CAD)
   │                                      │
   │  POST http://localhost:48765/execute  │
   │  {"code": "import k3; ..."}          │
   │─────────────────────────────────────►│
   │                                      │── exec(code)
   │                                      │── захват stdout/stderr
   │  {"stdout": "...", "success": true}  │
   │◄─────────────────────────────────────│
```

**Плюсы:** Быстро, надёжно, двусторонняя связь, простая реализация
**Минусы:** Нужен HTTP-сервер внутри CAD (Python 3.7 имеет `http.server` в стандартной библиотеке), только точка-точка

##### Вариант B — RabbitMQ (для нескольких CAD/распределённых систем)

```
                          ┌─────────────┐
                          │  RabbitMQ   │
                          │  (брокер)   │
                          └──────┬──────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                   │
              ▼                  ▼                   ▼
      ┌───────────────┐  ┌───────────────┐  ┌───────────────┐
      │  MCP Server   │  │  K3 Agent #1  │  │  K3 Agent #2  │
      │  (Producer/   │  │  (Consumer)   │  │  (Consumer)   │
      │   Consumer)   │  │  CAD #1       │  │  CAD #2       │
      └───────────────┘  └───────────────┘  └───────────────┘
```

**Схема обмена сообщениями:**

| Очередь | Назначение | Producer | Consumer |
|---------|-----------|----------|----------|
| `k3.requests` | Запросы на выполнение кода | MCP Server | K3 Agent |
| `k3.responses.{agent_id}` | Ответы от конкретного агента | K3 Agent | MCP Server |

**Плюсы:**
- **Масштабирование** — несколько CAD могут работать параллельно
- **Надёжность** — подтверждение доставки (ACK), очереди сохраняются при падении
- **Гибкость** — можно отправлять запросы конкретному CAD по `agent_id`
- **Асинхронность** — MCP сервер не блокируется, пока CAD выполняет код
- **Мониторинг** — RabbitMQ Management UI для отслеживания очередей

**Минусы:** Требуется установленный и настроенный RabbitMQ сервер, зависимость от `pika`

##### Вариант C — File-based JSON (без зависимостей)

```
MCP Server                          K3 Agent (в CAD)
   │                                      │
   │  пишет request_xxx.json              │
   │─────────────────────────────────────►│
   │                                      │── читает request
   │                                      │── exec(code)
   │                                      │── пишет response_xxx.json
   │◄─────────────────────────────────────│
   │  (опрос директории response/)        │
```

**Плюсы:** Не требует сетевого стека, 100% надёжность, нет внешних зависимостей
**Минусы:** Медленнее (опрос файловой системы), возможны конфликты при параллельных запросах

##### Вариант D — Named Pipes (Windows)

```
MCP Server                          K3 Agent (в CAD)
   │                                      │
   │  \\.\pipe\K3MCP                      │
   │─────────────────────────────────────►│
   │◄─────────────────────────────────────│
```

**Плюсы:** Быстро, нативно для Windows
**Минусы:** Сложнее в реализации, требует `pywin32`, только Windows

#### 4. K3 Agent

Скрипт, который запускается **внутри CAD K3** одним из двух способов:

1. **Напрямую** — через аргумент командной строки: `Mebel.exe -m:C:\path\to\k3_agent_tcp.py`
2. **Через `.mac` файл** — CAD K3 поддерживает формат `.mac` с тегами `<?python ... ?>` для встраивания Python-кода

Пример из существующего файла [`ExecPYC.mac`](Proto/Arline/ExecPYC.mac):

```python
<?python
import k3
import importlib
params = k3.getpar()
exec("import AReports.{} as foo".format(params[0][2:]))
imp = importlib.import_module(f'AReports.{params[0][2:]}')
importlib.reload(imp)
result = imp.main()
?>
```

### Полный цикл запроса

Рассмотрим на примере: пользователь просит "Получи материал профиля с UnitPos=67"

```
Шаг 1. SourceCraft Code Assistant → MCP Server (JSON-RPC):
       {
         "method": "tools/call",
         "params": {
           "name": "k3_execute",
           "arguments": {
             "code": "import k3; import mProfile;
                      k3.fltrparamobj(1,65);
                      k3.select(k3.k_partly,k3.k_all);
                      for i in range(int(k3.sysvar(61))):
                          p = k3.getselnum(i+1);
                          if k3.getattr(p,'UnitPos',-99) == 67:
                              with mProfile.PropertyProfile(p) as pr:
                                  print(pr.GetMater());
                              break;
                      k3.fltrparamobj(0)"
           }
         }
       }

Шаг 2. MCP Server → K3 Agent (TCP socket):
       socket.send('{"code": "import k3; import mProfile; ..."}')

Шаг 3. K3 Agent выполняет код внутри CAD:
       - import k3           ✓ (успешно, т.к. мы внутри CAD)
       - k3.fltrparamobj()   — фильтр по типу "профиль"
       - k3.select()         — выбор всех профилей
       - mProfile.PropertyProfile(p) — работа с профилем
       - pr.GetMater()       — получение материала
       - print()             — захватывается в stdout

Шаг 4. K3 Agent → MCP Server (TCP socket):
       socket.recv() →
       {
         "stdout": "{'mater': 25634, 'color_prof': 0}\n",
         "stderr": "",
         "success": true
       }

Шаг 5. MCP Server → SourceCraft Code Assistant (JSON-RPC):
       {
         "content": [{
           "type": "text",
           "text": "Материал профиля: ID=25634, цвет: 0"
         }]
       }
```

## Реализация

### Структура проекта

```
k3-mcp-server/
├── server.py              # MCP сервер (stdio transport + TCP-клиент)
├── k3_agent_tcp.py        # K3 Agent (TCP-сервер, запускается внутри CAD)
├── k3_tools.py            # Определения tools для MCP
├── requirements.txt       # Зависимости (пусто, только стандартная библиотека)
└── README.md
```

### MCP Server (`server.py`)

```python
#!/usr/bin/env python
"""
MCP Server для работы с CAD K3 через TCP-сокет.
CAD K3 запускается как TCP-сервер: Mebel.exe -m:k3_agent_tcp.py
"""
import json
import sys
import os
import socket

K3_HOST = os.environ.get("K3_HOST", "localhost")
K3_PORT = int(os.environ.get("K3_PORT", "9000"))
K3_TIMEOUT = int(os.environ.get("K3_TIMEOUT", "60"))


def send_to_k3(code: str, timeout: int = 30) -> dict:
    """Отправить код в CAD K3 через TCP-сокет и получить ответ."""
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.settimeout(timeout)
    try:
        sock.connect((K3_HOST, K3_PORT))

        # Отправить запрос
        request = json.dumps({"code": code})
        sock.sendall(request.encode())

        # Получить ответ (до разделителя \n)
        response_data = b""
        while True:
            chunk = sock.recv(4096)
            if not chunk:
                break
            response_data += chunk
            if b"\n" in chunk:
                break

        return json.loads(response_data.decode().strip())

    except socket.timeout:
        return {"error": "Timeout", "stderr": "CAD K3 не ответил за отведённое время"}
    except ConnectionRefusedError:
        return {"error": "CAD K3 не запущен",
                "stderr": "Запустите: Mebel.exe -m:k3_agent_tcp.py"}
    except Exception as e:
        return {"error": str(e)}
    finally:
        sock.close()


def handle_request(request):
    """Обработка JSON-RPC запросов MCP."""
    method = request.get("method")
    params = request.get("params", {})

    if method == "tools/list":
        return {
            "tools": [
                {
                    "name": "k3_execute",
                    "description": "Выполнить Python-код внутри CAD K3",
                    "inputSchema": {
                        "type": "object",
                        "properties": {
                            "code": {
                                "type": "string",
                                "description": "Python-код для выполнения внутри CAD"
                            },
                            "timeout": {
                                "type": "number",
                                "description": "Таймаут в секундах"
                            }
                        },
                        "required": ["code"]
                    }
                },
                {
                    "name": "k3_get_material",
                    "description": "Получить материал профиля по UnitPos",
                    "inputSchema": {
                        "type": "object",
                        "properties": {
                            "unitpos": {
                                "type": "number",
                                "description": "UnitPos профиля"
                            }
                        },
                        "required": ["unitpos"]
                    }
                },
                {
                    "name": "k3_get_order_info",
                    "description": "Получить информацию о заказе",
                    "inputSchema": {
                        "type": "object",
                        "properties": {
                            "field": {
                                "type": "string",
                                "description": "Поле заказа (Executor, Acceptor, AddInfo)"
                            }
                        },
                        "required": ["field"]
                    }
                }
            ]
        }

    elif method == "tools/call":
        tool_name = params.get("name")
        arguments = params.get("arguments", {})

        if tool_name == "k3_execute":
            result = send_to_k3(
                arguments["code"],
                arguments.get("timeout", 30)
            )
            return {"content": [{"type": "text", "text": json.dumps(result, ensure_ascii=False)}]}

        elif tool_name == "k3_get_material":
            code = f"""
import mProfile
import k3
k3.fltrparamobj(1, 65)
k3.select(k3.k_partly, k3.k_all)
for i in range(int(k3.sysvar(61))):
    p = k3.getselnum(i + 1)
    atr = k3.getattr(p, 'UnitPos', -99)
    if atr == {arguments['unitpos']}:
        with mProfile.PropertyProfile(p) as pr:
            print(pr.GetMater())
        break
k3.fltrparamobj(0)
"""
            result = send_to_k3(code, 30)
            return {"content": [{"type": "text", "text": json.dumps(result, ensure_ascii=False)}]}

        elif tool_name == "k3_get_order_info":
            code = f"""
import k3
print(k3.getorderinfo('{arguments['field']}'))
"""
            result = send_to_k3(code, 15)
            return {"content": [{"type": "text", "text": json.dumps(result, ensure_ascii=False)}]}

    return {"error": "Method not found"}


# Stdio transport loop
if __name__ == "__main__":
    print(f"K3 MCP Server запущен. CAD K3: {K3_HOST}:{K3_PORT}", file=sys.stderr)
    while True:
        line = sys.stdin.readline()
        if not line:
            break
        request = json.loads(line)
        response = handle_request(request)
        sys.stdout.write(json.dumps(response) + "\n")
        sys.stdout.flush()
```

### K3 Agent — TCP-сервер (`k3_agent_tcp.py`)

**Основной и рекомендуемый вариант.** Этот скрипт запускается внутри CAD K3 через `Mebel.exe -m:k3_agent_tcp.py` и открывает TCP-сокет для приёма запросов.

Основан на примере `test-server.py`, предоставленном разработчиком CAD K3.

```python
# -*- coding: utf-8 -*-
"""
k3_agent_tcp.py — K3 Agent (TCP-сервер).
Запускается внутри CAD K3: Mebel.exe -m:k3_agent_tcp.py
Слушает localhost:9000, выполняет Python-код, возвращает результат.

Основан на примере test-server.py (автор: Александр Драгункин)
"""
import socket
import sys
import json
import io
import contextlib
import traceback

# Добавляем Proto в sys.path (как в addFolderToSysPath.py)
sys.path.insert(0, k3.mpathexpand("<proto>"))

HOST = "localhost"
PORT = 9000


class StopServer(Exception):
    """Исключение для корректной остановки сервера."""
    pass


def execute_code(code: str) -> dict:
    """
    Выполнить Python-код внутри CAD K3.
    Захватывает stdout/stderr и возвращает результат.
    """
    stdout_cap = io.StringIO()
    stderr_cap = io.StringIO()

    try:
        with contextlib.redirect_stdout(stdout_cap), \
             contextlib.redirect_stderr(stderr_cap):
            exec(code)

        return {
            "stdout": stdout_cap.getvalue(),
            "stderr": stderr_cap.getvalue(),
            "success": True
        }
    except Exception:
        return {
            "stdout": stdout_cap.getvalue(),
            "stderr": stderr_cap.getvalue() + traceback.format_exc(),
            "success": False,
            "error": traceback.format_exc()
        }


def handle_request(data: bytes) -> bytes:
    """Обработать входящий запрос и вернуть ответ."""
    str_data = data.decode().strip()

    # Команда остановки сервера
    if str_data == "stop_server":
        raise StopServer()

    # Парсим JSON-запрос
    try:
        request = json.loads(str_data)
        code = request.get("code", "")
    except json.JSONDecodeError as e:
        return json.dumps({"error": f"Invalid JSON: {e}"}).encode()

    # Выполняем код
    result = execute_code(code)

    # Возвращаем результат с разделителем \n
    return (json.dumps(result, ensure_ascii=False) + "\n").encode()


# Создаём TCP-сокет
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

try:
    sock.bind((HOST, PORT))
    sock.listen()
    print(f"K3 MCP Agent запущен на {HOST}:{PORT}")
    print(f"Доступны модули: k3, mProfile, based_build, engine, ...")
    print(f"Отправьте 'stop_server' для завершения")

    while True:
        print("Ожидание соединения...")
        connection, client_address = sock.accept()
        try:
            print(f"Подключено: {client_address}")
            while True:
                data = connection.recv(65536)
                if not data:
                    break
                print(f"Получено: {data.decode()[:100]}...")
                response = handle_request(data)
                connection.sendall(response)
                print("Ответ отправлен")
        except StopServer:
            print("Получена команда остановки сервера")
            k3.new(k3.k_clear)
            k3.quit()
            break
        except Exception as e:
            print(f"Ошибка обработки: {e}")
        finally:
            connection.close()

finally:
    sock.close()
    print("K3 MCP Agent завершён")
```

## Развёртывание
### Шаг 1. Создать файлы MCP сервера

```bash
# Создать директорию для MCP сервера
mkdir c:\REPO\ARLINE\k3-mcp-server
cd c:\REPO\ARLINE\k3-mcp-server
```

Версия Python для MCP сервера **не важна** — подойдёт любая 3.x. Можно использовать уже существующие виртуальные окружения:

| Окружение | Описание |
|-----------|----------|
| `c:\VENV310\` | Python 3.10 (32-bit) |
| `c:\venv310-64\` | Python 3.10 (64-bit) |
| Любое другое 3.x | Системный Python или созданное `python -m venv venv` |

MCP сервер использует **только стандартную библиотеку** (`socket`, `json`, `sys`), поэтому дополнительные зависимости не требуются.
```

### Шаг 2. Создать файлы сервера

Создайте файлы в директории `k3-mcp-server/`:

| Файл | Описание |
|------|----------|
| `server.py` | MCP сервер (TCP-клиент, stdio transport) |
| `k3_agent_tcp.py` | K3 Agent (TCP-сервер, запускается внутри CAD) |

### Шаг 3. Запустить CAD K3 как TCP-сервер

**Способ A — прямой запуск с Python-скриптом (рекомендуемый):**

```bash
# Из командной строки:
"C:\ARL8\Bin\Mebel.exe" -m:C:\REPO\ARLINE\k3-mcp-server\k3_agent_tcp.py
```

**Способ B — через .mac файл:**

Создайте `startapp.mac`:
```python
<?python
import sys
sys.path.insert(0, r"C:\REPO\ARLINE\k3-mcp-server")
import k3_agent_tcp
?>
LoadOrder  last ;
```

Затем запустите:
```bash
"C:\ARL8\Bin\Mebel.exe" -m:startapp.mac
```

После запуска в консоли CAD появится сообщение:
```
K3 MCP Agent запущен на localhost:9000
Доступны модули: k3, mProfile, based_build, engine, ...
```

### Шаг 4. Настроить MCP сервер в вашем AI-ассистенте

Файл конфигурации зависит от ассистента (см. таблицу совместимости выше). Пример для SourceCraft Code Assistant:

**С системным Python (или любым venv):**
```json
{
  "mcpServers": {
    "k3-cad": {
      "command": "python",
      "args": [
        "c:/REPO/ARLINE/k3-mcp-server/server.py"
      ],
      "env": {
        "K3_HOST": "localhost",
        "K3_PORT": "9000",
        "K3_TIMEOUT": "60"
      },
      "disabled": false,
      "alwaysAllow": [],
      "disabledTools": []
    }
  }
}
```

**С конкретным виртуальным окружением (например, `c:\VENV310`):**
```json
{
  "mcpServers": {
    "k3-cad": {
      "command": "c:/VENV310/Scripts/python.exe",
      "args": [
        "c:/REPO/ARLINE/k3-mcp-server/server.py"
      ],
      "env": {
        "K3_HOST": "localhost",
        "K3_PORT": "9000",
        "K3_TIMEOUT": "60"
      },
      "disabled": false,
      "alwaysAllow": [],
      "disabledTools": []
    }
  }
}
```

### Шаг 5. Перезапустить AI-ассистента

После добавления конфигурации ассистент автоматически запустит MCP сервер и обнаружит доступные tools.

## Варианты использования

После настройки вы сможете делать такие запросы:

| Запрос | Действие |
|--------|----------|
| "Получи материал профиля UnitPos 67" | Выполнит `mProfile.PropertyProfile(p).GetMater()` внутри CAD |
| "Создай новый шкаф 800x600x2200" | Выполнит `based_build.Shkaf.Shkaf(800, 600, 2200)` |
| "Измени цвет фасада на ID 25634" | Выполнит `based_build.Fasad.Fasad().SetMater(25634)` |
| "Получи информацию о заказе" | Выполнит `k3.getorderinfo('Executor')` |
| "Найди все профили с UnitPos > 60" | Выполнит фильтрацию через `k3.fltrparamobj()` и `k3.selbyattr()` |
| "Выполни произвольный код" | Универсальный tool `k3_execute` для любых операций |

## Важные технические детали

### 1. Версия Python

**Внутри CAD K3** — **Python 3.7** (32-bit). Это видно из:
- [`addFolderToSysPath.py:84`](Proto/addFolderToSysPath.py:84): `sys.version_info >= (3, 7)`
- Путь к site-packages: `c:/VENV37aaa/Lib/site-packages`

**MCP сервер снаружи** — **любая версия Python 3.x**. Версия не важна, так как `server.py` использует только стандартную библиотеку (`socket`, `json`, `sys`). Можно запускать на системном Python без виртуального окружения.

### 2. Пути к модулям

CAD использует `k3.mpathexpand()` для получения путей:

| Идентификатор | Путь |
|---------------|------|
| `<app>` | `c:\ARL81\Bin` |
| `<proto>` | `c:\ARL81\DataApp\Data\PKM\Proto` |
| `<userproto>` | `c:\ARL81_UserData\Proto` |

Агентский скрипт должен добавить правильный путь в `sys.path`:
```python
sys.path.insert(0, k3.mpathexpand("<proto>"))
```

### 3. Один экземпляр CAD

TCP сервер на `localhost:9000` предполагает один запущенный CAD. Если CAD не запущен, MCP сервер вернёт ошибку `"CAD K3 не запущен"`.

### 4. Безопасность

TCP сервер не имеет аутентификации. Используйте только на `localhost`. Не открывайте порт 9000 наружу.

### 5. Сериализация результатов

Результаты `exec()` должны быть JSON-сериализуемыми. Объекты `k3.Group` и `k3.VarArray` нужно преобразовывать в строки/числа перед отправкой.

```python
# Неправильно:
result = k3.getselnum(1)  # Объект k3.Group

# Правильно:
result = str(k3.getselnum(1))
result_int = int(k3.sysvar(61))
```

### 6. Глобальное состояние CAD

CAD K3 хранит состояние в глобальных переменных (текущий заказ, выделенные объекты, активные фильтры). Агент не должен это состояние портить.

**Рекомендация:** после каждого запроса восстанавливать контекст, как в примере с `k3.fltrparamobj(0)` в `finally` блоке:

```python
try:
    k3.fltrparamobj(1, 65)  # Установить фильтр
    # ... работа с объектами ...
finally:
    k3.fltrparamobj(0)  # Сбросить фильтр
```

### 7. Обработка ошибок

Всегда оборачивайте выполнение кода в try/except и возвращайте traceback:

```python
try:
    exec(code)
except Exception as e:
    import traceback
    return {"error": str(e), "traceback": traceback.format_exc()}
```

## Альтернативный подход: пакетный режим CAD

Если CAD K3 поддерживает запуск из командной строки с выполнением Python-скрипта:

```python
import subprocess
import json
import tempfile


def execute_in_k3(code: str) -> dict:
    """Запустить CAD K3, выполнить код, получить результат."""
    with tempfile.NamedTemporaryFile(
        mode='w', suffix='.py', delete=False, encoding='utf-8'
    ) as f:
        f.write(f"""
import json, sys
sys.path.insert(0, r'c:\\REPO\\ARLINE\\Proto')

# Код пользователя
{code}

# Сохранить результат
result = {{"stdout": "...", "stderr": "..."}}
with open(r'{temp_result}', 'w') as rf:
    json.dump(result, rf)
""")
        script_path = f.name

    temp_result = tempfile.mktemp(suffix='.json')

    subprocess.run(
        ["k3.exe", "--script", script_path],
        cwd=r"c:\ARL81\Bin",
        timeout=60
    )

    with open(temp_result) as f:
        return json.load(f)
```

**Плюсы:** Не требует постоянно запущенного CAD
**Минусы:** Медленнее (каждый запуск CAD заново), сложнее отладка

## Сравнение вариантов IPC

| Характеристика | **TCP (рекомендуемый)** | HTTP | RabbitMQ | File-based | Named Pipes |
|---------------|------------------------|------|----------|------------|-------------|
| **Скорость** | ⚡ Быстро | ⚡ Быстро | ⚡ Быстро | 🐢 Медленно | ⚡ Быстро |
| **Надёжность** | ✅ Высокая | Средняя | ✅ Высокая (ACK, persistence) | ✅ Высокая | Средняя |
| **Масштабирование** | ❌ Нет | ❌ Нет | ✅ Несколько CAD | ❌ Нет | ❌ Нет |
| **Сложность** | 🟢 Очень низкая | 🟢 Низкая | 🟡 Средняя | 🟢 Низкая | 🔴 Высокая |
| **Зависимости** | **Нет** (стандартная библиотека) | `requests` | `pika` + RabbitMQ сервер | Нет | `pywin32` |
| **Запуск CAD** | `Mebel.exe -m:script.py` | Через `.mac` в UI | Через `.mac` в UI | Через `.mac` в UI | Через `.mac` в UI |
| **Асинхронность** | ❌ Синхронный | ❌ Синхронный | ✅ Асинхронный | ❌ Синхронный | ❌ Синхронный |
| **Мониторинг** | ❌ | ❌ | ✅ RabbitMQ Management UI | ❌ | ❌ |
| **Платформа** | Только Windows (CAD) | Кроссплатформа | Кроссплатформа | Кроссплатформа | Только Windows |

## Заключение

Рекомендуемая архитектура — **MCP сервер-прокси с TCP-сокетом**:

- **MCP сервер** (снаружи) — принимает запросы от AI-ассистента, отправляет команды через TCP-сокет
- **K3 Agent** (внутри CAD) — запускается как TCP-сервер через `Mebel.exe -m:k3_agent_tcp.py`, выполняет код с полным доступом к `k3`
- **Транспорт** — TCP-сокет (`localhost:9000`), только стандартная библиотека Python

Это единственный viable подход, так как `import k3` работает **только внутри процесса CAD K3**.

### Быстрый старт

```bash
# 1. Создать MCP сервер
mkdir c:\REPO\ARLINE\k3-mcp-server
cd c:\REPO\ARLINE\k3-mcp-server

# 2. Создать server.py и k3_agent_tcp.py (см. выше)

# 3. Запустить CAD K3 как TCP-сервер:
"C:\ARL8\Bin\Mebel.exe" -m:C:\REPO\ARLINE\k3-mcp-server\k3_agent_tcp.py

# 4. Добавить конфигурацию в mcp_settings.json вашего AI-ассистента

# 5. Готово! MCP сервер доступен для работы с CAD K3
```