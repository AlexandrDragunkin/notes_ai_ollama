# Финальный план завершения модуля mDPanel с учетом ограничений Python 3.7 32-битной среды Mebel.exe

## 1. Специфические ограничения среды

### 1.1. Ключевые технические ограничения
- **Python 3.7 32-битная** – строгое ограничение версии, избегать функций Python 3.8+
- **Среда выполнения** – код выполняется внутри приложения Mebel.exe через макроязык K3
- **Платформа** – эксклюзивно для Windows, без кроссплатформенности
- **Геометрические расчёты** – использовать встроенные функции K3 вместо внешних библиотек
- **Тестирование** – модульное тестирование с pytest 3.7, функциональное через RabbitMQ/Mebel.exe

### 1.2. Ограничения синтаксиса Python 3.7
- Избегать walrus operator `:=` (доступен с Python 3.8)
- Избегать `functools.cached_property` (доступен с Python 3.8)
- Использовать `typing.List` вместо `list` для аннотаций
- Использовать `typing.Dict` вместо `dict` для аннотаций
- Избегать позиционных-only параметров (доступны с Python 3.8)

## 2. Обновленная архитектура с учетом ограничений

### 2.1. PolyInfo - информация о полигоне (Python 3.7)
```python
# entities/poly_info.py
from dataclasses import dataclass
from typing import List, Tuple
from enum import Enum

class PathType(Enum):
    EXTERNAL = 1    # Внешний контур
    INTERNAL = 2    # Внутренний вырез
    MARKING = 3     # Линия маркировки

@dataclass
class PolyInfo:
    """Информация о результирующем полилайне панели (Python 3.7)"""
    n_paths: int
    paths: List['PathInfo']  # Python 3.7: использовать typing.List
    path_in: int = 0  # 0-с учетом кромки, 1-без учета
    
    def calculate_area(self) -> float:
        """Вычислить площадь полигона с использованием функций K3"""
        # Использовать k3.area или алгоритм shoelace formula на базе K3 функций
        # Python 3.7: избегать новых возможностей
        pass
```

### 2.2. PanelPropertyReader - сервис чтения (Python 3.7)
```python
# infrastructure/panel_property_reader.py
import k3
from typing import Dict, Any, List  # Python 3.7: явно импортировать typing
from ..entities.panel import Panel, PanelProperties

class PanelPropertyReader:
    """Сервис для чтения свойств панели из K3 объектов (Python 3.7)"""
    
    def read_from_k3_object(self, k3_object) -> Panel:
        """Прочитать панель из K3 объекта (совместимость с Python 3.7)"""
        # Python 3.7: использовать typing вместо встроенных аннотаций
        properties: PanelProperties = self._read_panel_properties(k3_object)
        return Panel(properties)
    
    def _read_panel_properties(self, k3_object) -> PanelProperties:
        """Прочитать свойства панели (Python 3.7)"""
        # Использовать getpan6par для чтения свойств
        # Python 3.7: избегать := оператора
        pass
```

## 3. Обновленная стратегия тестирования

### 3.1. Unit-тестирование (Python 3.7)
```python
# tests/test_poly_info.py
import pytest
from typing import List  # Python 3.7: явный импорт
from ..entities.poly_info import PolyInfo, PathType
from ..entities.path_info import PathInfo

class TestPolyInfo:
    """Тесты для класса PolyInfo (Python 3.7)"""
    
    @pytest.fixture
    def sample_poly_info(self):
        """Фикстура с примером PolyInfo (совместимость с Python 3.7)"""
        # Python 3.7: использовать typing.List вместо list
        paths: List[PathInfo] = [
            PathInfo(index=1, n_elems=4, elems=[], path_type=PathType.EXTERNAL),
            PathInfo(index=2, n_elems=2, elems=[], path_type=PathType.INTERNAL)
        ]
        return PolyInfo(n_paths=2, paths=paths, path_in=0)
```

### 3.2. Функциональное тестирование через RabbitMQ
```python
# tests/functional/test_rabbitmq_integration.py
import pika  # Совместимость с Python 3.7
import json
from typing import Dict, Any  # Python 3.7: явные импорты

class TestRabbitMQIntegration:
    """Тесты интеграции через RabbitMQ в среде Mebel.exe (Python 3.7)"""
    
    @pytest.mark.rabbitmq
    @pytest.mark.mebel_exe
    def test_panel_creation_via_message(self):
        """Тест создания панели через сообщение RabbitMQ"""
        # Python 3.7: использовать typing для аннотаций
        message: Dict[str, Any] = {
            "action": "create_panel",
            "properties": {
                "material": 502,
                "length": 2000.0,
                "width": 600.0
            }
        }
        
        # Отправка сообщения в Mebel.exe через RabbitMQ
        connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
        channel = connection.channel()
        channel.basic_publish(
            exchange='mebel_exchange',
            routing_key='panel.create',
            body=json.dumps(message)
        )
        
        # Ожидание ответа через журналы приложения
        log_entry = self._wait_for_log_entry('Panel created successfully')
        assert log_entry is not None
        
        connection.close()
```

### 3.3. Конфигурация pytest для Python 3.7
```ini
# pytest.ini для Python 3.7
[tool:pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = 
    -v
    --strict-markers
    --tb=short
markers =
    integration: integration tests
    slow: slow tests
    use_case: use case tests
    rabbitmq: tests requiring RabbitMQ
    mebel_exe: tests requiring Mebel.exe
```

## 4. Обновленный график реализации (8 недель)

### Неделя 1-2: Подготовка инфраструктуры (Python 3.7)
- [ ] Настройка среды разработки Python 3.7
- [ ] Создание структуры для новых сущностей с учетом typing
- [ ] Подготовка тестовой базы с pytest 3.7

### Неделя 3: Реализация сущностей (Python 3.7)  
- [ ] PolyInfo + тесты (совместимость с Python 3.7)
- [ ] PathInfo + тесты (совместимость с Python 3.7)
- [ ] ElemsInfo + тесты (совместимость с Python 3.7)
- [ ] FixInfoElems + тесты (совместимость с Python 3.7)

### Неделя 4: PanelPropertyReader (Python 3.7)
- [ ] Базовый сервис чтения с использованием K3 функций
- [ ] Парсеры для геометрической информации через K3
- [ ] Интеграционные тесты в среде Mebel.exe

### Неделя 5: K3Implementation (часть 1, Python 3.7)
- [ ] Метод get_panel с интеграцией RabbitMQ
- [ ] Метод list_panels через сообщения Mebel.exe
- [ ] Базовые тесты с учетом ограничений Python 3.7

### Неделя 6: K3Implementation (часть 2, Python 3.7)
- [ ] Метод save_panel с бэкапом через K3
- [ ] Методы для элементов с использованием K3 API
- [ ] Расширенные тесты через RabbitMQ

### Неделя 7: Оптимизация и тестирование (Python 3.7)
- [ ] Профилирование производительности в Mebel.exe
- [ ] Исправление багов совместимости с Python 3.7
- [ ] Достижение 90% покрытия с pytest 3.7

### Неделя 8: Документация и релиз (Python 3.7)
- [ ] Обновление документации с учетом ограничений
- [ ] Подготовка примеров использования для Mebel.exe
- [ ] Релиз v1.0.0 для среды Python 3.7

## 5. Критерии приемки (Python 3.7)

### 5.1. Функциональные требования
- ✅ Все 12 методов K3Implementation реализованы с учетом Python 3.7
- ✅ Поддержка чтения/записи через K3 API в Mebel.exe
- ✅ Обработка ошибок и исключений в среде Python 3.7
- ✅ Интеграция с legacy кодом PanelRectangle через K3

### 5.2. Качественные требования (Python 3.7)  
- ✅ Покрытие тестами ≥90% с pytest 3.7
- ✅ Документация для всех публичных методов с учетом ограничений
- ✅ Соответствие принципам чистой архитектуры в Python 3.7
- ✅ Производительность чтения панели < 1 секунда в Mebel.exe

### 5.3. Технические требования (Python 3.7)
- ✅ **Совместимость с Python 3.7 32-битная** – строгое соблюдение версии
- ✅ **Платформа Windows** – эксклюзивно для Windows в среде Mebel.exe
- ✅ **Интеграция с K3** – использование встроенных функций макроязыка K3
- ✅ **Тестирование** – модульное через pytest 3.7, функциональное через RabbitMQ
- ✅ **Готовность к использованию в продакшене** – внутри приложения Mebel.exe

## 6. Риски и митигация (Python 3.7)

### 6.1. Основные риски
1. **Ограничения Python 3.7** – отсутствие современных функций языка
   - Митигация: Использовать typing модуль, избегать новых синтаксических конструкций

2. **Интеграция с Mebel.exe** – зависимость от закрытой среды
   - Митигация: Тестирование через RabbitMQ, использование журналов приложения

3. **Геометрические расчеты** – невозможность использования внешних библиотек
   - Митигация: Использовать встроенные функции K3 для расчетов

### 6.2. Стратегия митигации
- **Поэтапное тестирование** – сначала модульные тесты, затем интеграционные
- **Локальное тестирование** – запуск тестов внутри среды Mebel.exe
- **Мониторинг журналов** – использование логов приложения для отладки
- **Резервные реализации** – альтернативные алгоритмы для геометрических расчетов

## 7. Заключение

Данный финальный план учитывает все специфические ограничения среды Python 3.7 32-битной в приложении Mebel.exe с интеграцией макроязыка K3. План обеспечивает:

1. **Полную совместимость** с Python 3.7 через использование typing модуля
2. **Интеграцию с Mebel.exe** через RabbitMQ и журналы приложения  
3. **Использование K3 функций** для геометрических расчетов вместо внешних библиотек
4. **Комплексное тестирование** с учетом ограничений среды выполнения

Реализация плана позволит создать надежный и поддерживаемый модуль mDPanel, готовый к использованию в продакшен-среде Mebel.exe с Python 3.7.