# Команды SourceCraft для рефакторинга Python пакета в K3 среде

## `/refactor-k3-architecture`
**argument-hint:** [scope: doors|hinges|facades|all]

Переработать архитектуру Python пакета для работы в K3 среде с учётом Clean Architecture принципов.

### Требования:
- Сохранить совместимость с 32-битной Python 3.7
- Изолировать K3 зависимости в инфраструктурном слое
- Использовать dataclasses для value objects
- Следовать принципу единственной ответственности

### Структура:
```
doorsys_k3/
├── domain/           # Чистая бизнес-логика
├── application/      # Use cases и координация  
├── infrastructure/   # K3 адаптеры
└── interfaces/      # K3 макросы
```

## `/extract-value-objects`
**argument-hint:** [from-file: путь]

Вынести Value Objects из существующего кода в отдельные dataclasses.

### Примеры для извлечения:
- `DoorDimensions` - ширина, высота, глубина
- `HingeConfig` - тип петли, углы, позиции  
- `DoorSpacing` - зазоры и нахлесты
- `FacadeSpec` - параметры фасада

### Требования:
- Использовать `@dataclass(frozen=True)` для иммутабельности
- Добавить методы валидации
- Вынести enum'ы в отдельные файлы

## `/create-k3-adapter`
**argument-hint:** [entity-name: Door|Facade|Hinge]

Создать адаптер между доменной сущностью и K3 объектом.

### Шаблон адаптера:
```python
class K3EntityAdapter:
    def __init__(self, entity: Entity, k3_object):
        self.entity = entity
        self.k3_obj = k3_object
    
    def apply_to_k3(self):
        """Применить параметры entity к K3 объекту"""
        pass
    
    def read_from_k3(self):
        """Прочитать параметры из K3 объекта в entity"""
        pass
```

## `/refactor-god-class`
**argument-hint:** [class-name: StandartDoor|StraightDoor]

Разделить God Class на специализированные компоненты.

### Компоненты для выделения:
- `DoorSpecification` - параметры двери
- `DoorCalculator` - расчётные методы  
- `DoorValidator` - проверки корректности
- `DoorBuilder` - построение K3 объектов

### Принципы:
- Каждый класс ≤ 200 строк
- Одна ответственность на класс
- Чёткие интерфейсы между компонентами

## `/create-use-case`
**argument-hint:** [scenario: create-door|build-facade|add-hinges]

Создать Use Case для конкретного бизнес-сценария.

### Структура Use Case:
```python
class CreateDoorUseCase:
    def __init__(self, repo: DoorRepository, validator: DoorValidator):
        self.repo = repo
        self.validator = validator
    
    def execute(self, dto: DoorDTO) -> Door:
        # 1. Валидация входных данных
        # 2. Создание доменной сущности
        # 3. Вызов бизнес-правил
        # 4. Сохранение через репозиторий
        # 5. Возврат результата
```

## `/extract-k3-utils`
**argument-hint:** [category: attributes|db|objects]

Вынести K3-специфичные утилиты в отдельный сервис.

### Утилиты для извлечения:
- Работа с атрибутами K3
- Чтение переменных из базы данных  
- Создание и настройка K3 объектов
- Обработка ошибок K3 API

## `/implement-door-factory`
**argument-hint:** [door-type: straight|standard|wingline]

Реализовать фабрику для создания различных типов дверей.

### Паттерн фабрики:
```python
class DoorFactory:
    @staticmethod
    def create(door_type: str, config: dict) -> Door:
        if door_type == "straight":
            return StraightDoorFactory.create(config)
        elif door_type == "standard":
            return StandardDoorFactory.create(config)
        # ...
```

## `/create-integration-test`
**argument-hint:** [component: entity|service|adapter]

Создать тест для проверки работы компонента в K3 среде.

### Тестовый подход:
- Mock K3 зависимостей где возможно
- Интеграционные тесты для адаптеров
- Проверка корректности преобразований
- Тестирование бизнес-правил изолированно

## `/update-changelog`
**argument-hint:** [version: 0.6.0|patch|minor]

Обновить CHANGELOG.md в соответствии с Conventional Commits.

### Формат записей:
```markdown
## [0.6.0] - 2024-01-15
### Added
- Новая архитектура доменного слоя
- Адаптеры для K3 объектов

### Changed
- Рефакторинг StandartDoor на компоненты

### Fixed
- Исправление расчёта позиций петель
```

## `/verify-k3-compatibility`
**argument-hint:** [check-type: imports|api|performance]

Проверить совместимость рефакторинга с K3 фреймворком.

### Проверки:
- Отсутствие запрещённых импортов (multiprocessing)
- Корректность использования K3 API
- Производительность в 32-битной среде
- Сохранение существующих интерфейсов

## Convention Commits для проекта:

### Примеры коммитов:
```
feat(doors): добавить доменную модель DoorEntity
refactor(standart-door): разделить на DoorBuilder и DoorValidator  
fix(hinges): исправить расчёт позиций для HINV петель
docs(architecture): добавить документацию новой структуры
test(adapters): добавить интеграционные тесты K3 адаптеров
chore(deps): обновить версию K3 utils до 1.2.0
```

### Области действия (scope):
- `doors` - основные сущности дверей
- `facades` - работа с фасадами  
- `hinges` - петли и фурнитура
- `architecture` - изменения структуры пакета
- `k3` - K3-специфичные компоненты
- `tests` - тестовое покрытие

Каждая команда фокусируется на одной конкретной задаче рефакторинга и включает чёткие требования к реализации.