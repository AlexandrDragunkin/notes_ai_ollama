# Стратегия тестирования модуля mDPanel (Python 3.7 32-битная среда Mebel.exe)

## 1. Обзор тестовой стратегии

### 1.1. Цели тестирования
- **Обеспечить 90%+ покрытие кода**
- **Поддержать чистую архитектуру модуля**
- **Обеспечить регрессионную защиту**
- **Подготовить модуль к продакшену**

### 1.2. Уровни тестирования (Python 3.7)
| Уровень | Цель | Покрытие | Инструменты |
|---------|------|----------|-------------|
| **Unit** | Проверка отдельных компонентов | 70% | pytest 3.7, unittest.mock |
| **Integration** | Взаимодействие компонентов | 20% | pytest 3.7, тестовые K3 файлы |
| **Use-case** | Бизнес-сценарии | 10% | pytest 3.7, RabbitMQ/Mebel.exe |
| **Regression** | Защита от отката | 100% | Локальный CI/CD |

**Ограничения среды:**
- **Python 3.7 32-битная** – строгое соблюдение версии
- **Среда Mebel.exe** – выполнение внутри приложения
- **Тестирование** – модульное через pytest 3.7, функциональное через RabbitMQ
- **Результаты** – через журналы приложения или ответные сообщения

## 2. Unit-тестирование

### 2.1. Структура unit-тестов

#### 2.1.1. Тесты для новых сущностей
```python
# tests/test_poly_info.py
import pytest
from ..entities.poly_info import PolyInfo, PathType
from ..entities.path_info import PathInfo

class TestPolyInfo:
    """Тесты для класса PolyInfo (Python 3.7)"""
    
    @pytest.fixture
    def sample_poly_info(self):
        """Фикстура с примером PolyInfo (совместимость с Python 3.7)"""
        # Python 3.7: использовать typing.List вместо list
        from typing import List
        paths = List[PathInfo] = [
            PathInfo(index=1, n_elems=4, elems=[], path_type=PathType.EXTERNAL),
            PathInfo(index=2, n_elems=2, elems=[], path_type=PathType.INTERNAL)
        ]
        return PolyInfo(n_paths=2, paths=paths, path_in=0)
    
    def test_poly_info_creation(self, sample_poly_info):
        """Тест создания PolyInfo"""
        assert sample_poly_info.n_paths == 2
        assert len(sample_poly_info.paths) == 2
        assert sample_poly_info.path_in == 0
    
    def test_calculate_area(self, sample_poly_info):
        """Тест вычисления площади"""
        # Мокировать геометрические расчеты
        area = sample_poly_info.calculate_area()
        assert isinstance(area, float)
        assert area >= 0
```

#### 2.1.2. Тесты для PanelPropertyReader
```python
# tests/test_panel_property_reader.py
import pytest
from unittest.mock import Mock, patch
from ..infrastructure.panel_property_reader import PanelPropertyReader
from ..entities.panel import Panel, PanelProperties

class TestPanelPropertyReader:
    """Тесты для сервиса чтения свойств панели (Python 3.7)"""
    
    @pytest.fixture
    def mock_k3_object(self):
        """Фикстура для мока K3 объекта (совместимость с Python 3.7)"""
        mock_obj = Mock()
        mock_obj.getattr.return_value = "010000"  # Valid panel type
        # Python 3.7: избегать новых возможностей unittest.mock
        return mock_obj
    
    @pytest.fixture
    def panel_reader(self):
        """Фикстура для PanelPropertyReader"""
        return PanelPropertyReader()
    
    @patch('k3.getpan6par')
    def test_read_panel_properties(self, mock_getpan6par, panel_reader, mock_k3_object):
        """Тест чтения свойств панели"""
        # Настроить мок для возврата корректных данных
        mock_getpan6par.return_value = 1
        mock_getpan6par.side_effect = self._mock_getpan6par_side_effect
        
        properties = panel_reader._read_panel_properties(mock_k3_object)
        
        assert isinstance(properties, PanelProperties)
        assert properties.material.id == 502
        assert properties.dimensions.length == 2000.0
    
    def _mock_getpan6par_side_effect(self, code, array):
        """Мок для функции getpan6par"""
        if code == 2:  # Material
            array[0].value = 502
        elif code == 11:  # Dimensions
            array[1].value = 2000.0  # length
            array[2].value = 600.0   # width
        return 1
```

### 2.2. Моки и фикстуры

#### 2.2.1. Расширение существующих фикстур
```python
# tests/conftest.py (расширение)
@pytest.fixture
def sample_poly_info():
    """Фикстура с примером PolyInfo"""
    from ..entities.poly_info import PolyInfo, PathType
    from ..entities.path_info import PathInfo
    
    paths = [
        PathInfo(index=1, n_elems=4, elems=[], path_type=PathType.EXTERNAL),
        PathInfo(index=2, n_elems=2, elems=[], path_type=PathType.INTERNAL)
    ]
    return PolyInfo(n_paths=2, paths=paths, path_in=0)

@pytest.fixture
def mock_panel_reader():
    """Фикстура для мока PanelPropertyReader"""
    from unittest.mock import Mock
    reader = Mock()
    reader.read_from_k3_object.return_value = sample_panel()  # Использовать существующую фикстуру
    return reader
```

## 3. Интеграционное тестирование

### 3.1. Тесты взаимодействия компонентов

#### 3.1.1. Тест полного цикла работы с панелью
```python
# tests/test_integration.py
import pytest
from pathlib import Path
from ..interfaces.k3_implementation import K3Implementation
from ..entities.panel import PanelProperties, PanelMaterial, PanelDimensions

class TestK3Integration:
    """Интеграционные тесты для K3Implementation (Python 3.7)"""
    
    @pytest.fixture
    def k3_implementation(self):
        """Фикстура для K3Implementation"""
        return K3Implementation()
    
    @pytest.fixture
    def sample_panel_properties(self):
        """Фикстура с примером свойств панели"""
        material = PanelMaterial(id=502, name="ДСП", thickness=16.0)
        dimensions = PanelDimensions(length=2000.0, width=600.0, thickness=16.0)
        return PanelProperties(
            material=material,
            dimensions=dimensions,
            panel_type=PanelType.SHELF,
            form=PanelForm.RECTANGULAR
        )
    
    def test_create_and_read_panel_workflow(self, k3_implementation, sample_panel_properties):
        """Тест полного цикла создания и чтения панели"""
        # 1. Создать панель в K3
        panel_handle = k3_implementation.create_panel(sample_panel_properties)
        assert panel_handle is not None
        
        # 2. Прочитать свойства обратно
        properties = k3_implementation.get_panel_properties(panel_handle)
        assert properties.material.id == sample_panel_properties.material.id
        assert properties.dimensions.length == sample_panel_properties.dimensions.length
        
        # 3. Удалить панель
        k3_implementation.delete_panel(panel_handle)
```

### 3.2. Тесты с реальными K3 файлами

#### 3.2.1. Тестовые данные
```python
# tests/test_data/__init__.py
"""Тестовые данные для интеграционных тестов"""

TEST_K3_FILES = {
    'simple_panel': Path('tests/test_data/simple_panel.k3'),
    'complex_panel': Path('tests/test_data/complex_panel.k3'),
    'panel_with_cuts': Path('tests/test_data/panel_with_cuts.k3')
}

class TestK3Files:
    """Тесты работы с реальными K3 файлами (Python 3.7)"""
    
    @pytest.mark.integration
    def test_read_simple_panel_from_file(self, k3_implementation):
        """Тест чтения простой панели из файла (Python 3.7)"""
        test_file = TEST_K3_FILES['simple_panel']
        
        # Python 3.7: использовать pathlib.Path совместимый с 3.7
        from pathlib import Path
        test_path = Path(str(test_file))  # Совместимость с Python 3.7
        
        # Предполагаем, что метод get_panel_from_file будет реализован
        panel = k3_implementation.get_panel_from_file(test_path)
        
        assert panel is not None
        assert panel.properties.dimensions.length > 0
        assert panel.properties.dimensions.width > 0
```

## 4. Use-case тестирование

### 4.1. Бизнес-сценарии

#### 4.1.1. Сценарий создания сложной панели
```python
# tests/test_use_cases.py (расширение)
class TestComplexPanelUseCases:
    """Тесты сложных use-case сценариев (Python 3.7 + RabbitMQ)"""
    
    def test_create_panel_with_multiple_elements(self, k3_implementation):
        """Тест создания панели с множеством элементов (интеграция с RabbitMQ)"""
        # Python 3.7: использовать совместимый синтаксис
        
        # 1. Отправка сообщения через RabbitMQ в Mebel.exe
        rabbitmq_message = self._create_rabbitmq_panel_message()
        response = self._send_rabbitmq_message(rabbitmq_message)
        
        # 2. Ожидание обработки в Mebel.exe
        panel_id = self._wait_for_panel_creation(response)
        
        # 3. Чтение результата через журналы приложения
        log_data = self._read_mebel_logs(panel_id)
        
        # 4. Проверка результатов через K3Implementation
        panel_handle = k3_implementation.get_panel_handle(panel_id)
        full_panel = k3_implementation.get_panel(panel_handle)
        
        # 5. Проверить результаты
        assert full_panel is not None
        assert len(full_panel.slots) > 0
        assert len(full_panel.bands) > 0
        
        # 6. Очистка через RabbitMQ
        self._send_cleanup_message(panel_id)
    
    def _create_complex_panel_properties(self):
        """Создать сложные свойства панели"""
        # Реализация создания тестовых данных
        pass
```

### 4.2. Тесты обработки ошибок

#### 4.2.1. Сценарии ошибок
```python
# tests/test_error_handling.py
import pytest
from ..exceptions.panel_exceptions import PanelError, PanelCreationError

class TestErrorHandling:
    """Тесты обработки ошибок (Python 3.7 + Mebel.exe)"""
    
    def test_invalid_panel_handle(self, k3_implementation):
        """Тест обработки невалидного handle панели"""
        invalid_handle = "invalid_handle"
        
        with pytest.raises(PanelError):
            k3_implementation.get_panel_properties(invalid_handle)
    
    def test_corrupted_k3_file(self, k3_implementation):
        """Тест обработки поврежденного K3 файла (интеграция с Mebel.exe)"""
        corrupted_file = Path('tests/test_data/corrupted.k3')
        
        # Отправка сообщения с поврежденным файлом через RabbitMQ
        error_message = self._create_error_message(corrupted_file)
        response = self._send_rabbitmq_message(error_message)
        
        # Проверка ошибки через журналы Mebel.exe
        error_log = self._read_error_logs(response.message_id)
        
        with pytest.raises(PanelError):
            k3_implementation.get_panel_from_file(corrupted_file)
```

## 5. Регрессионное тестирование

### 5.1. Локальная конфигурация тестирования (Python 3.7)

#### 5.1.1. Конфигурация pytest для Python 3.7
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

# Запуск тестов внутри Mebel.exe:
# python -m pytest tests/ -v -m "not rabbitmq and not mebel_exe"
```

#### 5.1.2. Функциональное тестирование через RabbitMQ
```python
# tests/functional/test_rabbitmq_integration.py
import pika  # Совместимость с Python 3.7
import json
from typing import Dict, Any

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

### 5.2. Мониторинг покрытия

#### 5.2.1. Целевые метрики покрытия
```python
# pytest.ini (расширение)
[tool:pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = 
    -v
    --cov=.
    --cov-report=term-missing
    --cov-report=html
    --cov-fail-under=90
    --strict-markers
markers =
    integration: integration tests
    slow: slow tests
    use_case: use case tests
```

## 6. План внедрения тестирования

### 6.1. Поэтапное внедрение

**Этап 1 (Неделя 1-2): Базовое unit-тестирование**
- Тесты для существующих сущностей
- Моки для K3 модуля
- 60% покрытие базовой функциональности

**Этап 2 (Неделя 3-4): Интеграционное тестирование**
- Тесты взаимодействия компонентов
- Тесты с тестовыми K3 файлами
- 75% общее покрытие

**Этап 3 (Неделя 5-6): Use-case тестирование**
- Бизнес-сценарии
- Тесты обработки ошибок
- 85% общее покрытие

**Этап 4 (Неделя 7-8): Оптимизация и CI/CD**
- Регрессионные тесты
- Настройка CI/CD
- 90%+ целевое покрытие

### 6.2. Критерии успеха тестирования

- ✅ **Покрытие кода ≥90%**
- ✅ **Все unit-тесты проходят**
- ✅ **Интеграционные тесты с реальными данными**
- ✅ **Use-case тесты покрывают ключевые сценарии**
- ✅ **CI/CD pipeline настроен и работает**
- ✅ **Обработка ошибок тестируется полностью**

## 7. Заключение

Данная стратегия тестирования обеспечивает комплексный подход к обеспечению качества модуля mDPanel. Поэтапное внедрение тестов с четкими критериями успеха позволит создать надежный и тестируемый код, готовый к использованию в продакшен-среде.