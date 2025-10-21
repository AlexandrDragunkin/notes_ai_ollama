# Детальный технический план реализации модуля mDPanel

## 1. Анализ текущего состояния - ЗАВЕРШЕНО ✅

### 1.1. Выявленные проблемы
- **12 методов в [`K3Implementation`](../PRESTIGE/prestige_user_proto_81/src/Proto/mDPanel/interfaces/k3_implementation.py) имеют заглушки**
- **Отсутствуют ключевые сущности для работы с геометрией панели**
- **Логика чтения свойств сосредоточена в legacy классе [`PanelRectangle`](../PRESTIGE/prestige_user_proto_81/src/Proto/drawprof/mPanel.py:1145)**

### 1.2. Существующие активы
- ✅ Полная архитектура чистой архитектуры
- ✅ Комплексные тесты для сущностей и сервисов
- ✅ Моки для K3 модуля в [`conftest.py`](../PRESTIGE/prestige_user_proto_81/src/Proto/mDPanel/conftest.py:33)
- ✅ Готовые use-case сценарии

## 2. Проектирование недостающих классов

### 2.1. PolyInfo - информация о полигоне
**Срок: 3 дня**

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
    """Информация о результирующем полилайне панели"""
    n_paths: int
    paths: List['PathInfo']
    path_in: int = 0  # 0-с учетом кромки, 1-без учета
    
    def calculate_area(self) -> float:
        """Вычислить площадь полигона"""
        # Использовать алгоритм shoelace formula
        pass
        
    def get_bounding_box(self) -> Tuple[float, float, float, float]:
        """Получить габаритный прямоугольник"""
        pass
```

### 2.2. PathInfo - информация о контуре
**Срок: 2 дня**

```python
# entities/path_info.py
from dataclasses import dataclass
from typing import List, Optional

@dataclass
class PathInfo:
    """Информация по контуру панели"""
    index: int
    n_elems: int
    elems: List['ElemsInfo']
    depth: Optional[float] = None      # Глубина для глухих вырезов
    cut_side: Optional[str] = None     # Сторона положения
    str_info: str = ""                 # Комментарий контура
    
    @property
    def is_std_dimchain(self) -> bool:
        """Формировать стандартную размерную сетку"""
        return "NOT_DIMDRAWS" not in self.str_info and "NoteForDraw" not in self.str_info
```

### 2.3. ElemsInfo - информация об элементах контура
**Срок: 3 дня**

```python
# entities/elems_info.py
from dataclasses import dataclass
from typing import List, Optional
from enum import Enum

class ElementType(Enum):
    POINT = 0
    LINE = 1
    ARC = 2
    SPLINE = 3

@dataclass
class ElemsInfo:
    """Информация по элементу контура"""
    id_poly: int
    id_line: int
    type_elem: ElementType
    geo_info: List[float]              # Геометрическая информация
    length: Optional[float] = None
    band: 'BandInfoElems' = None       # Информация по кромке
    fixes: 'FixInfoElems' = None       # Информация по крепежу
```

### 2.4. FixInfoElems - информация по крепежу
**Срок: 2 дня**

```python
# entities/fix_info_elems.py
from dataclasses import dataclass
from typing import List

@dataclass
class FixInfo:
    """Информация по крепежу на линии"""
    fix_type: int                      # Тип крепежа из таблицы
    num_line: int                      # Номер линии крепежа
    shift_line: float                  # Сдвиг линии от начала элемента
    orders: int                        # Номер правила установки
    b_mask: int                        # Битовые маски
    length: float                      # Длина линии крепежа

@dataclass
class FixInfoElems:
    """Информация по крепежу на элементе контура"""
    count: int
    fix_info: List[FixInfo]
```

## 3. Создание сервиса PanelPropertyReader

### 3.1. Архитектура сервиса
**Срок: 5 дней**

```python
# infrastructure/panel_property_reader.py
import k3
from pathlib import Path
from typing import Dict, Any, List
from ..entities.panel import Panel, PanelProperties
from ..entities.poly_info import PolyInfo
from ..entities.path_info import PathInfo
from ..entities.elems_info import ElemsInfo, ElementType
from ..entities.fix_info_elems import FixInfoElems, FixInfo

class PanelPropertyReader:
    """Сервис для чтения свойств панели из K3 объектов"""
    
    def __init__(self):
        self._a_pan = None
        
    def read_from_k3_object(self, k3_object) -> Panel:
        """Прочитать панель из K3 объекта"""
        # 1. Инициализировать панель
        a_pan = self._panel_init(k3_object)
        
        try:
            # 2. Прочитать основные свойства
            properties = self._read_panel_properties(k3_object)
            
            # 3. Прочитать геометрическую информацию
            poly_info = self._read_poly_info(k3_object)
            
            # 4. Создать доменную сущность
            panel = Panel(properties)
            panel.poly_info = poly_info
            
            return panel
            
        finally:
            self._clear_structure(a_pan)
    
    def _read_poly_info(self, k3_object) -> PolyInfo:
        """Прочитать информацию о полигоне"""
        poly_info = PolyInfo()
        
        # Реализация на основе PanelRectangle.getPanelPathInfo
        # Использовать getpan6par с кодами 31, 32, 33, 34
        
        return poly_info
```

### 3.2. Интеграция с K3Implementation
**Срок: 2 дня**

```python
# interfaces/k3_implementation.py (модификация)
class K3Implementation(K3Interface):
    def __init__(self):
        self._ap_pan = None
        self.panel_reader = PanelPropertyReader()  # Добавить сервис
    
    def get_panel(self, panel_handle: Any) -> Panel:
        """Получить полную модель панели"""
        try:
            return self.panel_reader.read_from_k3_object(panel_handle)
        except Exception as e:
            raise PanelError(f"Ошибка чтения панели: {str(e)}")
```

## 4. Реализация методов K3Implementation

### 4.1. Приоритетная очередь реализации

**Высокий приоритет (Неделя 4):**
1. `get_panel` - полное чтение панели
2. `save_panel` - сохранение изменений
3. `list_panels` - перечисление панелей

**Средний приоритет (Неделя 5):**
4. `add_slot`, `get_slots` - работа с пропилами
5. `add_cut`, `get_cuts` - работа с врезками
6. `add_band`, `get_bands` - работа с кромками

**Низкий приоритет (Неделя 6):**
7. Остальные методы для крепежа, отделки, фрезеровки

### 4.2. Детальная реализация методов

**Метод `get_panel` (3 дня):**
```python
def get_panel(self, panel_handle: Any) -> Panel:
    """Получить полную модель панели из K3"""
    k3_module = self._get_k3_module()
    
    try:
        # 1. Проверить валидность объекта
        if not self._is_valid_panel(panel_handle):
            raise PanelError("Невалидный объект панели")
            
        # 2. Использовать PanelPropertyReader
        panel = self.panel_reader.read_from_k3_object(panel_handle)
        
        # 3. Дополнить информацией об элементах
        panel.slots = self._get_slots_from_k3(panel_handle)
        panel.cuts = self._get_cuts_from_k3(panel_handle)
        # ... остальные элементы
        
        return panel
        
    except Exception as e:
        raise PanelError(f"Ошибка получения панели: {str(e)}")
```

**Метод `save_panel` (4 дня):**
```python
def save_panel(self, panel: Panel, file_path: Path) -> None:
    """Сохранить панель в K3 файл"""
    try:
        # 1. Создать бэкап существующего файла
        self._create_backup(file_path)
        
        # 2. Сериализовать панель в K3 структуру
        k3_data = self._serialize_panel_to_k3(panel)
        
        # 3. Записать в файл
        self._write_k3_file(file_path, k3_data)
        
        # 4. Валидировать записанные данные
        self._validate_saved_panel(file_path, panel)
        
    except Exception as e:
        # 5. Восстановить из бэкапа при ошибке
        self._restore_from_backup(file_path)
        raise PanelError(f"Ошибка сохранения панели: {str(e)}")
```

## 5. Стратегия тестирования

### 5.1. Структура тестового покрытия

**Unit-тесты (70% покрытия):**
```
tests/
├── test_poly_info.py          # Тесты PolyInfo
├── test_path_info.py          # Тесты PathInfo  
├── test_elems_info.py         # Тесты ElemsInfo
├── test_fix_info_elems.py     # Тесты FixInfoElems
├── test_panel_property_reader.py # Тесты сервиса чтения
└── test_k3_implementation.py  # Тесты реализации
```

**Интеграционные тесты (20% покрытия):**
```python
# tests/test_integration.py
class TestK3Integration:
    def test_full_panel_workflow(self):
        """Полный workflow работы с панелью"""
        # 1. Создать панель
        # 2. Добавить элементы
        # 3. Сохранить в K3
        # 4. Прочитать обратно
        # 5. Сравнить результаты
        pass
```

**Use-case тесты (10% покрытия):**
```python
# tests/test_use_cases.py (расширить)
class TestAdvancedUseCases:
    def test_panel_with_complex_geometry(self):
        """Тест панели со сложной геометрией"""
        pass
        
    def test_panel_with_multiple_elements(self):
        """Тест панели с множеством элементов"""
        pass
```

### 5.2. CI/CD конфигурация
```yaml
# .github/workflows/ci.yml
name: mDPanel CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run tests with coverage
        run: |
          cd src/Proto/mDPanel
          python -m pytest tests/ -v --cov=. --cov-report=xml
      - name: Upload coverage
        uses: codecov/codecov-action@v1
```

## 6. График реализации (8 недель)

### Неделя 1-2: Подготовка инфраструктуры
- [ ] Настройка CI/CD pipeline
- [ ] Создание структуры для новых сущностей
- [ ] Подготовка тестовой базы

### Неделя 3: Реализация сущностей  
- [ ] PolyInfo + тесты
- [ ] PathInfo + тесты
- [ ] ElemsInfo + тесты
- [ ] FixInfoElems + тесты

### Неделя 4: PanelPropertyReader
- [ ] Базовый сервис чтения
- [ ] Парсеры для геометрической информации
- [ ] Интеграционные тесты

### Неделя 5: K3Implementation (часть 1)
- [ ] Метод get_panel
- [ ] Метод list_panels  
- [ ] Базовые тесты

### Неделя 6: K3Implementation (часть 2)
- [ ] Метод save_panel
- [ ] Методы для элементов
- [ ] Расширенные тесты

### Неделя 7: Оптимизация и тестирование
- [ ] Профилирование производительности
- [ ] Исправление багов
- [ ] Достижение 90% покрытия

### Неделя 8: Документация и релиз
- [ ] Обновление документации
- [ ] Подготовка примеров использования
- [ ] Релиз v1.0.0

## 7. Критерии приемки

### 7.1. Функциональные требования
- ✅ Все 12 методов K3Implementation реализованы
- ✅ Поддержка чтения/записи K3 файлов
- ✅ Обработка ошибок и исключений
- ✅ Интеграция с legacy кодом PanelRectangle

### 7.2. Качественные требования  
- ✅ Покрытие тестами ≥90%
- ✅ Документация для всех публичных методов
- ✅ Соответствие принципам чистой архитектуры
- ✅ Производительность чтения панели < 1 секунда

### 7.3. Технические требования
- ✅ Совместимость с Python 3.8+
- ✅ Поддержка Windows/Linux
- ✅ Интеграция с существующим CI/CD
- ✅ Готовность к использованию в продакшене

## 8. Заключение

Данный план обеспечивает систематический подход к завершению модуля mDPanel с четкими сроками, критериями качества и стратегией тестирования. Реализация плана позволит создать надежный и поддерживаемый модуль для работы с мебельными панелями в системе K3.