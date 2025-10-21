# Комплексный план завершения модуля mDPanel (Python 3.7 32-битная среда Mebel.exe)

## 1. Анализ текущего состояния

### 1.1. Существующая архитектура
Модуль mDPanel имеет чистую архитектуру с четким разделением на слои:

**✅ Реализовано полностью:**
- **Entities**: [`Panel`](../PRESTIGE/prestige_user_proto_81/src/Proto/mDPanel/entities/panel.py:53), [`Slot`](../PRESTIGE/prestige_user_proto_81/src/Proto/mDPanel/entities/slot.py), [`CutLine`](../PRESTIGE/prestige_user_proto_81/src/Proto/mDPanel/entities/cutline.py), [`Band`](../PRESTIGE/prestige_user_proto_81/src/Proto/mDPanel/entities/band.py), [`FixLine`](../PRESTIGE/prestige_user_proto_81/src/Proto/mDPanel/entities/fixline.py), [`Decorate`](../PRESTIGE/prestige_user_proto_81/src/Proto/mDPanel/entities/decorate.py), [`Butt`](../PRESTIGE/prestige_user_proto_81/src/Proto/mDPanel/entities/butt.py), [`Mill`](../PRESTIGE/prestige_user_proto_81/src/Proto/mDPanel/entities/mill.py)
- **Interfaces**: [`K3Interface`](../PRESTIGE/prestige_user_proto_81/src/Proto/mDPanel/interfaces/k3_interface.py:17) (абстрактный)
- **Use Cases**: [`panel_use_cases.py`](../PRESTIGE/prestige_user_proto_81/src/Proto/mDPanel/use_cases/panel_use_cases.py)
- **Services**: [`PanelService`](../PRESTIGE/prestige_user_proto_81/src/Proto/mDPanel/services/panel_service.py)
- **Tests**: Полный набор тестов для сущностей и сервисов

**⚠️ Требует реализации:**
- **K3Implementation**: Многие методы помечены "Реализация будет добавлена позже" (строки 189, 194, 199, 204, 209, 214, 219, 224, 229, 234, 239, 244, 249, 254)
- **Отсутствующие сущности**: `PolyInfo`, `PathInfo`, `ElemsInfo`, `FixInfoElems`

### 1.2. Код legacy для интеграции
Из [`mPanel.py`](../PRESTIGE/prestige_user_proto_81/src/Proto/drawprof/mPanel.py) необходимо извлечь:
- Класс [`PanelRectangle`](../PRESTIGE/prestige_user_proto_81/src/Proto/drawprof/mPanel.py:1145) с логикой чтения свойств панели
- Классы [`PolyInfo`](../PRESTIGE/prestige_user_proto_81/src/Proto/drawprof/mPanel.py:63), [`PathInfo`](../PRESTIGE/prestige_user_proto_81/src/Proto/drawprof/mPanel.py:171), [`ElemsInfo`](../PRESTIGE/prestige_user_proto_81/src/Proto/drawprof/mPanel.py:313), [`FixInfoElems`](../PRESTIGE/prestige_user_proto_81/src/Proto/drawprof/mPanel.py:531)

## 2. Проектирование недостающих классов

### 2.1. PolyInfo - информация о полигоне (контур панели)
```python
@dataclass
class PolyInfo:
    points: List[Tuple[float, float]]  # Координаты точек
    closed: bool                       # Замкнутый контур
    n_paths: int                       # Количество путей
    paths: List['PathInfo']            # Список путей
    
    def area(self) -> float:
        """Вычислить площадь полигона с использованием функций K3"""
        # Использовать k3.area или алгоритм shoelace formula на базе K3
        pass
        
    def perimeter(self) -> float:
        """Вычислить периметр с использованием функций K3"""
        # Использовать k3.distance для расчета длин отрезков
        pass
        
    def contains(self, point: Tuple[float, float]) -> bool:
        """Проверить, содержит ли полигон точку"""
        pass
```

### 2.2. PathInfo - описание пути (контур выреза/вставки)
```python
@dataclass
class PathInfo:
    segments: List['Segment']          # Сегменты пути
    index: int                         # Индекс пути
    n_elems: int                       # Количество элементов
    elems: List['ElemsInfo']           # Элементы пути
    depth: float                       # Глубина (для глухих вырезов)
    cut_side: str                      # Сторона положения
    
    def length(self) -> float:
        """Вычислить длину пути с использованием функций K3"""
        # Использовать k3.distance для расчета длин элементов
        pass
        
    def is_closed(self) -> bool:
        """Проверить, замкнут ли путь"""
        pass
```

### 2.3. ElemsInfo - сводка всех элементов
```python
@dataclass
class ElemsInfo:
    cuts: List['CutLine']              # Вырезы
    inserts: List['Insert']            # Вставки
    edges: List['Band']                # Кромки
    fasteners: List['FixLine']         # Крепеж
    
    def total_volume(self) -> float:
        """Вычислить общий объем элементов с использованием функций K3"""
        # Использовать k3.volume или расчет на основе площади и толщины
        pass
        
    def validate(self) -> bool:
        """Проверить валидность элементов"""
        pass
```

### 2.4. FixInfoElems - параметры крепежных элементов
```python
@dataclass
class FixInfoElems:
    fixes: List['Fix']                 # Список крепежей
    count: int                         # Количество
    
    def group_by_type(self) -> Dict[str, List['Fix']]:
        """Сгруппировать крепежи по типу"""
        pass
        
    def export_k3(self) -> Dict[str, Any]:
        """Экспортировать в формат K3"""
        pass
```

## 3. Создание сервиса PanelPropertyReader

### 3.1. Архитектура сервиса
```python
class PanelPropertyReader:
    """Сервис для чтения свойств панели из K3 файлов"""
    
    def read_from_k3(self, k3_file: Path) -> Panel:
        """Прочитать панель из K3 файла"""
        # 1. Открыть K3 файл
        # 2. Найти секцию PanelRectangle
        # 3. Использовать парсеры для PolyInfo, PathInfo, ElemsInfo, FixInfoElems
        # 4. Преобразовать в доменную сущность Panel
        pass
        
    def parse_poly_info(self, k3_data: Dict) -> PolyInfo:
        """Парсить информацию о полигоне"""
        pass
        
    def parse_path_info(self, k3_data: Dict) -> PathInfo:
        """Парсить информацию о пути"""
        pass
        
    def parse_elems_info(self, k3_data: Dict) -> ElemsInfo:
        """Парсить информацию об элементах"""
        pass
        
    def parse_fix_info(self, k3_data: Dict) -> FixInfoElems:
        """Парсить информацию о крепежах"""
        pass
```

### 3.2. Интеграция с K3Implementation
```python
class K3Implementation(K3Interface):
    def get_panel(self, panel_id: str) -> Panel:
        """Получить полную модель панели"""
        # Вызвать PanelPropertyReader.read_from_k3
        return self.panel_reader.read_from_k3(panel_id)
```

## 4. Реализация методов K3Implementation

### 4.1. Приоритетные методы для реализации

| Метод | Статус | Сложность | Приоритет |
|-------|--------|-----------|-----------|
| `get_panel` | ❌ Заглушка | Высокая | 🔴 Высокий |
| `save_panel` | ❌ Заглушка | Высокая | 🔴 Высокий |
| `list_panels` | ❌ Заглушка | Средняя | 🟡 Средний |
| `delete_panel` | ✅ Реализован | Низкая | 🟢 Низкий |
| `add_slot` | ❌ Заглушка | Средняя | 🟡 Средний |
| `get_slots` | ❌ Заглушка | Средняя | 🟡 Средний |
| `add_cut` | ❌ Заглушка | Средняя | 🟡 Средний |
| `get_cuts` | ❌ Заглушка | Средняя | 🟡 Средний |

### 4.2. Детальный план реализации

**Метод `get_panel` (строки 80-114):**
1. Интегрировать `PanelPropertyReader`
2. Реализовать парсинг K3 структуры
3. Обработать ошибки и исключения
4. Написать unit-тесты

**Метод `save_panel` (отсутствует в интерфейсе):**
1. Добавить в `K3Interface`
2. Реализовать сериализацию `Panel` → K3 структура
3. Реализовать запись в файл с бэкапом
4. Написать интеграционные тесты

**Методы для элементов (строки 187-255):**
1. Реализовать базовые операции CRUD
2. Интегрировать с существующими сущностями
3. Написать тесты для каждого метода

## 5. Стратегия тестирования

### 5.1. Уровни тестирования

**Unit-тесты (90% покрытие):**
- Тестирование отдельных классов и парсеров
- Моки для K3 модуля
- Тесты валидации данных

**Интеграционные тесты:**
- Взаимодействие `K3Implementation` ↔ `PanelPropertyReader` ↔ файловая система
- Тесты чтения/записи реальных K3 файлов

**Use-case тесты:**
- Полные бизнес-сценарии
- Цепочки операций (создание → модификация → сохранение)

**Регрессионные тесты:**
- Защита от отката функциональности
- Автоматический запуск при каждом коммите

### 5.2. Структура тестов
```
tests/
├── test_entities.py (расширить для новых сущностей)
├── test_property_reader.py (новый)
├── test_services.py (расширить)
├── test_k3_implementation.py (новый)
└── test_use_cases.py (расширить)
```

## 6. План документации

### 6.1. Документы для обновления
- **README.md** - общее описание модуля
- **API документация** - автогенерация из doc-строк
- **Guidelines** - руководство по добавлению новых методов
- **Changelog** - история изменений

### 6.2. Примеры использования
Создать файл `examples/advanced_usage.py` с примерами:
- Чтение панели из K3 файла
- Модификация свойств
- Сохранение изменений
- Обработка ошибок

## 7. Реалистичный график (8-недельный спринт)

### Неделя 1: Подготовка и инфраструктура
- Настройка CI/CD pipeline
- Создание задач в трекере
- Добавление базовых тестов для новой функциональности

### Неделя 2: Дизайн и реализация сущностей
- Проектирование `PolyInfo`, `PathInfo`, `ElemsInfo`, `FixInfoElems`
- Написание unit-тестов для новых сущностей
- Рефакторинг legacy кода

### Неделя 3: Сервис PanelPropertyReader
- Создание сервиса чтения свойств
- Интеграция парсеров из `PanelRectangle`
- Тестирование парсинга K3 данных

### Неделя 4: Реализация K3 методов (часть 1)
- `get_panel`, `list_panels`
- Интеграционные тесты
- Обновление `K3Interface`

### Неделя 5: Реализация K3 методов (часть 2)
- `save_panel`, `delete_panel`
- Тесты регрессии
- Реализация бэкапов/откатов

### Неделя 6: Расширенные операции
- `update_fasteners`, `calculate_metrics`
- Use-case тесты
- Примеры в документации

### Неделя 7: Полное тестирование
- Запуск полного набора тестов
- Достижение ≥90% coverage
- Исправление багов и оптимизация

### Неделя 8: Документация и релиз
- Завершение документации
- Подготовка релиза v1.0.0
- Интеграция в основной проект

## 8. Риски и митигация

### 8.1. Основные риски
1. **Неоднородный формат K3 файлов**
   - Митигация: Покрыть тестами парсинг разных вариантов
   
2. **Сложные геометрические расчеты**
   - Митигация: Использовать библиотеку `shapely`

3. **Регрессия старого кода**
   - Митигация: Обязательный запуск полного набора тестов

### 8.2. Критерии успеха
- ✅ Все методы `K3Implementation` реализованы
- ✅ Покрытие тестами ≥90%
- ✅ Интеграция с legacy кодом завершена
- ✅ Документация актуальна и полна
- ✅ Модуль готов к использованию в продакшене

## 9. Заключение

Данный план обеспечивает систематический подход к завершению модуля mDPanel с учетом существующей архитектуры и требований к интеграции с legacy кодом. Реализация плана позволит создать надежный, тестируемый и документированный модуль, готовый к использованию в основном проекте.