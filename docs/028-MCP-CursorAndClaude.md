# MCP-серветы в контексте использования Cursor и Claude

## 🤖 Что такое MCP-серверы?

**MCP (Model Context Protocol)** — это протокол, который позволяет AI-ассистентам подключаться к внешним инструментам и ресурсам для расширения своих возможностей.

### 🔧 Основные функции MCP:

- **Доступ к локальным файлам** и проектам
- **Интеграция с IDE** и редакторами кода
- **Подключение к базам данных**
- **Доступ к внутренним API**
- **Работа с файловой системой**

## 🚀 Как MCP используется в Cursor и Claude

### В Cursor:
```
Cursor ↔ MCP-сервер ↔ Ваш локальный проект
```

Cursor использует MCP для:
- **Чтения структуры проекта**
- **Понимания контекста кода**
- **Редактирования файлов "на месте"**
- **Запуска команд в терминале**

### В Claude:
```
Claude ↔ MCP-сервер ↔ Ваши данные и инструменты
```

Claude через MCP может:
- **Анализировать локальные документы**
- **Работать с вашими базами данных**
- **Использовать специфичные для компании данные**

## 🛠 Настройка MCP-сервера

### 1. Установка MCP-сервера:

```bash
# Установка через npm
npm install -g @modelcontextprotocol/server

# Или через Docker
docker run -p 3000:3000 modelcontextprotocol/server
```

### 2. Конфигурация сервера:

```json
{
  "name": "My Development Server",
  "capabilities": {
    "fileSystem": true,
    "terminal": true,
    "database": true
  },
  "workspace": "/path/to/your/project"
}
```

## 🎯 Практическое применение в вашем workflow

### Для Python + k3 API:

```python
# Cursor может напрямую работать с вашими скриптами через MCP
# и понимать структуру вашей библиотеки k3

# Пример запроса через Cursor:
"Проанализируй скрипт generate_furniture.py и 
оптимизируй работу с k3 API для генерации 1000 деталей"
```

### Для Битрикс:

```php
// MCP позволяет Cursor напрямую работать с файлами Битрикс
// и понимать структуру компонентов

// Пример:
"Объясни, как работает компонент catalog.section 
в контексте нашего проекта мебели"
```

### Для 1С:

```
# Через MCP Claude может анализировать 
# конфигурации и обработки 1С

Запрос: "Проанализируй обработку ImportOrders.epf 
и предложи оптимизации для работы с большими объемами"
```

## 🔧 Преимущества MCP в вашем случае:

### 1. **Интеграция с закрытой библиотекой k3:**
```python
# Cursor понимает структуру вашей библиотеки через MCP
# и может предлагать улучшения, основанные на реальном коде

"Как оптимизировать функцию create_3d_model() 
в вашей библиотеке k3 для работы с параметрическими моделями?"
```

### 2. **Работа с MSSQL/MS Access:**
```sql
-- MCP позволяет Claude напрямую анализировать 
-- структуру ваших баз данных

Запрос: "Проанализируй таблицы Materials и Orders 
и предложи оптимизацию запросов для генерации отчетов"
```

### 3. **Генерация отчетов XLSX/PDF:**
```python
# Cursor может видеть ваши шаблоны отчетов через MCP
# и предлагать улучшения

"Оптимизируй скрипт generate_report.py для 
быстрой генерации PDF-отчетов по 1000 заказам"
```

## 🚀 Пример настройки для вашего workflow:

### Конфигурационный файл MCP:

```yaml
# mcp-config.yaml
server:
  name: "Furniture Production Server"
  port: 3001
  
capabilities:
  fileSystem: true
  terminal: true
  database: 
    mssql: true
    access: true
  
workspace:
  root: "/projects/furniture-cad"
  exclude: ["node_modules", ".git", "temp"]
  
integrations:
  k3_library: "/lib/k3"
  bitrix: "/www/bitrix"
  _1c: "/1c/Configurator"
  
databases:
  production_db:
    type: "mssql"
    connection: "server=localhost;database=FurnitureDB"
  orders_db:
    type: "access"
    path: "/data/orders.accdb"
```

### Запуск MCP-сервера:

```bash
# Запуск с вашей конфигурацией
mcp-server --config mcp-config.yaml

# Сервер будет доступен по адресу: http://localhost:3001
```

## 🎯 Интеграция с AI-ассистентами:

### В Cursor:
1. Откройте настройки Cursor
2. Добавьте MCP-сервер: `http://localhost:3001`
3. Cursor автоматически получит доступ к вашему проекту

### В Claude (через API):
```python
import requests

# Подключение к MCP-серверу
mcp_url = "http://localhost:3001"
headers = {"Authorization": "Bearer YOUR_MCP_TOKEN"}

# Запрос к Claude с контекстом из вашего проекта
response = requests.post(
    f"{mcp_url}/claude/query",
    headers=headers,
    json={
        "prompt": "Проанализируй скрипты в /python/cad/ и предложи оптимизации",
        "context": "project_files"
    }
)
```

## 📊 Преимущества для вашего бизнеса:

### 1. **Повышение производительности:**
- Cursor может напрямую оптимизировать ваш код
- Claude анализирует реальные данные, а не абстракции

### 2. **Безопасность:**
- Данные остаются локально
- Нет необходимости передавать конфиденциальную информацию во внешние API

### 3. **Точность:**
- AI-ассистенты работают с актуальным кодом
- Понимают специфику вашей библиотеки k3

### 4. **Автоматизация:**
```bash
# Пример автоматизации через MCP
mcp-cli execute "python /scripts/optimize_models.py" --watch
```

## 🛠 Рекомендации по внедрению:

### Этап 1: Базовая настройка
1. Установите MCP-сервер
2. Настройте доступ к вашим проектам
3. Подключите Cursor и Claude

### Этап 2: Интеграция с Python + k3
```python
# Пример использования в ваших скриптах
from mcp_client import MCPClient

client = MCPClient("http://localhost:3001")
client.analyze_code("/python/cad/model_generator.py")
```

### Этап 3: Интеграция с Битрикс и 1С
```php
// PHP-интеграция с MCP
$mcp = new MCPClient("http://localhost:3001");
$mcp->analyze_component("catalog.furniture");
```

## 🎯 Будущие возможности:

### Скоро в разработке:
- **Реальное время** анализа изменений в коде
- **Автоматические рефакторинги** по расписанию
- **Интеграция с CI/CD** для автоматической проверки кода
- **Многопользовательская среда** для командной работы

# Работа с библиотекой k3 через AI-ассистенты

## 📋 Анализ вашего модуля mProfile

Отличный пример сложного модуля, демонстрирующего работу с библиотекой k3 для создания 3D-моделей мебели. Давайте разберём, как AI-ассистенты могут помочь в разработке и оптимизации такого кода.

## 🎯 Основные компоненты модуля:

### 1. **PropertyProfile** - класс для работы со свойствами профиля
```python
# Основные функции:
- SetMater() / GetMater() - работа с материалами
- SetTrim() / GetTrim() - подрезки профиля  
- SetPosition() / GetPosition() - позиционирование
- SetShape() / GetShape() - форма профиля
- SetDecorate() / GetDecorate() - отделка профиля
```

### 2. **ProfileData** - класс данных профиля
```python
# Содержит все параметры профиля:
- Материалы и цвета
- Подрезки и углы
- Сдвиги и симметрии
- Отделка и декор
```

### 3. **Profile** - основной класс профиля (наследник FurnObject)
```python
# Создание и отрисовка профилей в 3D-пространстве
- Draw() - отрисовка профиля
- _AssignAttributes() - присвоение атрибутов
```

## 🤖 Как AI-ассистенты могут помочь:

### **Cursor** для работы с этим кодом:

#### 1. **Объяснение сложных функций:**
```python
# Выделите функцию angle_z_calculate и спросите:
"Объясни, как работает эта функция вычисления угла между векторами"
```

#### 2. **Оптимизация кода:**
```python
# Выделите метод Draw() и спросите:
"Как оптимизировать этот метод для работы с тысячами профилей?"
```

#### 3. **Генерация документации:**
```python
# Выделите класс PropertyProfile и спросите:
"Создай подробную документацию с примерами использования для этого класса"
```

### **Claude** для архитектурного анализа:

#### 1. **Анализ производительности:**
```
Проанализируй этот код и предложи улучшения для работы с большими объемами данных (1000+ профилей)
```

#### 2. **Архитектурные улучшения:**
```
Как улучшить архитектуру этого модуля для лучшей масштабируемости и поддержки?
```

## 🚀 Практические примеры использования:

### 1. **Оптимизация PropertyProfile:**

#### Проблема: Множество вызовов k3.setprof6par/k3.getprof6par

#### Решение через Cursor:
```python
# Запрос: "Оптимизируй класс PropertyProfile для уменьшения 
# количества вызовов API k3 и улучшения производительности"

# Cursor может предложить:
class OptimizedPropertyProfile:
    def __init__(self, profile=None):
        self._cache = {}  # Кэширование свойств
        self._batch_operations = []  # Пакетные операции
        
    def batch_set_properties(self, properties_dict):
        """Установка всех свойств за одну операцию"""
        # Реализация пакетной установки свойств
        pass
```

### 2. **Улучшение ProfileData:**

#### Через Claude:
```
Как улучшить класс ProfileData для лучшей валидации данных 
и предотвращения ошибок при создании профилей?
```

#### Результат:
```python
class EnhancedProfileData:
    def __init__(self):
        self._validators = {
            'id_material': self._validate_material,
            'length': self._validate_length,
            'angles': self._validate_angles
        }
    
    def _validate_material(self, material_id):
        """Проверка существования материала в базе данных"""
        if not k3.priceinfo(material_id, 'MATNAME', 0):
            raise ValueError(f"Материал {material_id} не найден в справочнике")
        return material_id
```

### 3. **Генерация тестов:**

#### Через Cursor:
```python
# Выделите модуль и спросите:
"Создай unit-тесты для класса Profile с использованием pytest"
```

#### Результат:
```python
import pytest
from mProfile import Profile, ProfileData

class TestProfile:
    def test_profile_creation(self):
        """Тест создания профиля с валидными данными"""
        data = ProfileData()
        data.SetMater(12345)  # Валидный ID материала
        data.SetLength(1000)  # Длина 1000 мм
        
        profile = Profile(data=data)
        assert profile.data.GetMater() == 12345
        assert profile.data.GetLength() == 1000
    
    def test_invalid_material(self):
        """Тест обработки невалидного материала"""
        data = ProfileData()
        with pytest.raises(ValueError):
            data.SetMater(-1)  # Невалидный ID
```

## 🛠 Интеграция с MCP-сервером:

### Конфигурация для вашего проекта:

```yaml
# mcp-config.yaml
server:
  name: "k3 Development Server"
  port: 3001

capabilities:
  fileSystem: true
  terminal: true
  database: 
    mssql: true

workspace:
  root: "/projects/mProfile"
  exclude: ["__pycache__", ".git"]

integrations:
  k3_library: "/lib/k3"
  cad_models: "/models"
  
databases:
  materials_db:
    type: "mssql"
    connection: "server=localhost;database=MaterialsDB"
```

### Использование в разработке:

#### 1. **Cursor с MCP:**
```python
# Cursor может напрямую анализировать вашу библиотеку k3
# и предлагать улучшения, основанные на реальной структуре API

# Запрос: "Проанализируй использование k3.getprof6par 
# и предложи более эффективные альтернативы"
```

#### 2. **Claude с доступом к данным:**
```
Проанализируй структуру классов в mProfile и предложи 
рефакторинг для лучшей интеграции с системой управления материалами
```

## 🎯 Рекомендации по улучшению:

### 1. **Кэширование свойств:**
```python
class CachedPropertyProfile(PropertyProfile):
    """Версия с кэшированием для улучшения производительности"""
    
    def __init__(self, profile=None):
        super().__init__(profile)
        self._property_cache = {}
        
    def GetMater(self):
        """Кэшированное получение материала"""
        cache_key = 'material'
        if cache_key not in self._property_cache:
            self._property_cache[cache_key] = super().GetMater()
        return self._property_cache[cache_key]
```

### 2. **Пакетная обработка:**
```python
class BatchProfileProcessor:
    """Обработчик пакетов профилей для массовой генерации"""
    
    def __init__(self):
        self.profiles = []
        
    def add_profile(self, profile_data):
        """Добавить профиль в пакет"""
        self.profiles.append(profile_data)
        
    def process_batch(self):
        """Обработать все профили пакетом"""
        # Оптимизированная обработка множества профилей
        pass
```

### 3. **Валидация данных:**
```python
def validate_profile_data(profile_data):
    """Комплексная валидация данных профиля"""
    validators = [
        validate_material_exists,
        validate_length_positive,
        validate_angles_range,
        validate_decorations_valid
    ]
    
    for validator in validators:
        try:
            validator(profile_data)
        except ValidationError as e:
            logger.error(f"Ошибка валидации: {e}")
            raise
```

## 🚀 Пример комплексного запроса:

### В Cursor:
```
Проанализируй модуль mProfile и предложи:
1. Оптимизацию для работы с 1000+ профилями одновременно
2. Улучшение обработки ошибок и логирования
3. Интеграцию с системой управления материалами MSSQL
4. Генерацию документации в формате Sphinx
```

### В Claude:
```
Проанализируй архитектуру модуля mProfile и предложи:
1. Как улучшить масштабируемость для промышленного производства
2. Как интегрировать с системами PLM и ERP
3. Какие паттерны проектирования лучше применить
4. Как улучшить тестируемость кода
```

