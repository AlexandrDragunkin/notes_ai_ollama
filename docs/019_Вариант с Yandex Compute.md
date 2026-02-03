Вариант с Yandex Compute Cloud виртуальными машинами.

## Архитектура решения на Yandex Compute Cloud

### 1. Создание базового образа ВМ

#### Cloud-init скрипт для подготовки ВМ:
```yaml
#cloud-config
package_update: true
package_upgrade: true

runcmd:
  # Установка зависимостей
  - powershell -Command "Invoke-WebRequest -Uri https://download.microsoft.com/download/5/F/0/5F0F6A85-3FBA-42F3-B2F3-5F7B2468C3FA/ENU/amd64/accessdatabaseengine_X64.exe -OutFile C:\access.exe"
  - powershell -Command "Start-Process C:\access.exe -ArgumentList '/quiet' -Wait"
  
  # Установка .NET 4.6.1
  - powershell -Command "Invoke-WebRequest -Uri https://download.microsoft.com/download/E/4/1/E4173890-A24A-4936-9FC9-AF930FE3FA40/NDP461-KB3102436-x86-x64-AllOS-ENU.exe -OutFile C:\dotnet.exe"
  - powershell -Command "Start-Process C:\dotnet.exe -ArgumentList '/quiet', '/norestart' -Wait"
  
  # Установка Python 3.7.9 32-bit
  - powershell -Command "Invoke-WebRequest -Uri https://www.python.org/ftp/python/3.7.9/python-3.7.9.exe -OutFile C:\python.exe"
  - powershell -Command "Start-Process C:\python.exe -ArgumentList '/quiet', 'InstallAllUsers=1', 'PrependPath=1' -Wait"
  
  # Копирование приложения (из Object Storage)
  - powershell -Command "Invoke-WebRequest -Uri https://storage.yandexcloud.net/k3-app/mebel.exe -OutFile C:\app\mebel.exe"
  - powershell -Command "Invoke-WebRequest -Uri https://storage.yandexcloud.net/k3-app/k3_library.zip -OutFile C:\app\k3_library.zip"
  - powershell -Command "Expand-Archive -Path C:\app\k3_library.zip -DestinationPath C:\app\"
  
  # Настройка окружения
  - powershell -Command "[Environment]::SetEnvironmentVariable('K3_APP_PATH', 'C:\app', 'Machine')"

power_state:
  mode: poweroff
```

### 2. Создание Instance Group для тестирования

```bash
# Создание группы виртуальных машин
yc compute instance-group create \
  --name k3-test-group \
  --folder-id b1gxxxxxxxxxxxxxx \
  --platform standard-v3 \
  --zone ru-central1-a \
  --scale-size 10 \
  --metadata-from-file user-data=cloud-config.yaml \
  --template-labels environment=test,app=k3 \
  --service-account-name k3-test-sa \
  --network-interface subnets=default-ru-central1-a,ipv4-address=auto,nat-ip-version=ipv4 \
  --template-resources memory=4G,cores=2,core-fraction=100 \
  --template-boot-disk-type=network-ssd,size=50,image-id=fd8vmcue7a............
```

### 3. Система управления тестами

#### Центральный диспетчер тестов (отдельная ВМ):
```python
# test_dispatcher.py
import yandex.cloud.compute.v1.instance_service_pb2 as instance_service
import yandex.cloud.compute.v1.instance_service_pb2_grpc as instance_service_grpc
import grpc
import json
from concurrent.futures import ThreadPoolExecutor

class TestDispatcher:
    def __init__(self):
        self.channel = grpc.secure_channel('compute.api.cloud.yandex.net:443')
        self.instance_service = instance_service_grpc.InstanceServiceStub(self.channel)
    
    def run_test_suite(self, test_suite_name, instance_count=10):
        # Запуск ВМ с тестами
        for i in range(instance_count):
            self.start_test_instance(test_suite_name, i)
    
    def start_test_instance(self, test_suite_name, instance_index):
        # Запуск конкретной ВМ с тестом
        pass
```

### 4. Интеграция с базами данных

#### Настройка подключения к Managed Databases:
```yaml
# database_config.yaml
mssql:
  host: rc1a-xxxxxxxxxxxx.db.yandex.net
  port: 1433
  database: k3_tests
  username: tester
  password: ${DB_PASSWORD}

postgresql:
  host: rc1b-xxxxxxxxxxxx.db.yandex.net  
  port: 6432
  database: k3_tests
  username: tester
  password: ${DB_PASSWORD}
```

### 5. Система мониторинга и логирования

#### Настройка Yandex Monitoring:
```bash
# Создание дашборда для мониторинга тестов
yc monitoring dashboard create \
  --name k3-test-dashboard \
  --description "K3 Furniture Testing Dashboard"
```

#### Сбор логов через Yandex Logging:
```python
# log_collector.py
import yandex.cloud.logging.v1.log_reading_service_pb2 as log_reading
import yandex.cloud.logging.v1.log_reading_service_pb2_grpc as log_reading_grpc

def collect_test_logs(instance_id):
    # Сбор и анализ логов тестов
    pass
```

### 6. Автоматизация через Yandex Cloud Functions

#### Функция для запуска тестовой сессии:
```python
# start_test_session.py
import json
import yandex.cloud.serverless.functions.v1.function_service_pb2 as function_service

def handler(event, context):
    event_data = json.loads(event['body'])
    
    test_suite = event_data['test_suite']
    parallel_count = event_data.get('parallel_count', 10)
    
    # Запуск тестовой группы
    result = start_test_group(test_suite, parallel_count)
    
    return {
        'statusCode': 200,
        'body': json.dumps(result)
    }
```

### 7. Пример скрипта тестирования на ВМ

```python
# test_runner.ps1
param(
    [string]$TestSuite,
    [string]$DatabaseType,
    [int]$WorkerId
)

# Настройка окружения
$env:PATH = "C:\Python37;" + $env:PATH

# Запуск приложения с тестами
Start-Process -FilePath "C:\app\mebel.exe" -ArgumentList "--test-suite", $TestSuite, "--db", $DatabaseType, "--worker", $WorkerId -Wait

# Отправка результатов
Invoke-WebRequest -Uri "https://functions.yandexcloud.net/dashboard/test-results" -Method POST -Body @{worker=$WorkerId; results="results.xml"}
```

### 8. Стоимость и оптимизация

#### Расчет стоимости для 100 параллельных тестов:
```bash
# Конфигурация ВМ: 2 vCPU, 4GB RAM, 50GB SSD
yc compute instance-type list --format json | jq '.[] | select(.cores == 2 and .memory == 4)'
```

#### Оптимизация затрат:
- Использование preemptible instances
- Автоматическое выключение ВМ после тестов
- Оптимальный выбор зон доступности

### 9. Преимущества этого подхода:

1. **Полная совместимость** с Windows-приложением
2. **Гибкое масштабирование** от 1 до 100+ ВМ
3. **Интеграция** со всеми сервисами Yandex Cloud
4. **Простота отладки** - прямой доступ к ВМ
5. **Надежность** - изоляция тестовых окружений

### 10. Next Steps:

1. Создайте базовый образ ВМ с вашим приложением
2. Протестируйте на 2-3 ВМ
3. Настройте мониторинг и логирование
4. Автоматизируйте запуск тестовых сессий

Этот подход даст вам полный контроль над тестовой средой и позволит легко масштабироваться под любые объемы тестирования.