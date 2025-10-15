# Архитектурный подход к рефакторингу большого пакета Door_user

Отлично! Исходя из предыдущего анализа и выбранной модели **Clean Architecture + DDD**, вот конкретные предложения по рефакторингу вашего кода дверей:

## 1. Новая структура пакета

```
doorsys/
├── core/                           # Чистая бизнес-логика
│   ├── domain/
│   │   ├── entities/              # Сущности (Door, Facade, Hinge)
│   │   ├── value_objects/         # Value Objects (Dimensions, HingeType)
│   │   ├── services/              # Domain Services
│   │   └── repositories.py        # Интерфейсы репозиториев
│   └── application/
│       ├── use_cases/             # Use Cases
│       ├── dto/                   # Data Transfer Objects
│       └── services/              # Application Services
├── infrastructure/                 # Реализации инфраструктуры
│   ├── persistence/               # Работа с БД, K3
│   ├── external/                  # Внешние сервисы
│   └── config/                    # Конфигурация
├── interfaces/                    # Внешние интерфейсы
│   ├── api/                       # REST API
│   ├── cli/                       # Командная строка
│   └── web/                       # Web интерфейс
└── shared/                        # Общие утилиты
    ├── utils/
    └── exceptions/
```

## 2. Конкретные преобразования классов

### 2.1. Вынести Value Objects

```python
# core/domain/value_objects/door_config.py
from dataclasses import dataclass
from enum import Enum
from typing import Tuple

class Hinge4Type(Enum):
    INV = 1    # Накладная
    HINV = 2   # Полунакладная
    INS = 3    # Вкладная

@dataclass(frozen=True)
class HingeOpenType:
    htype: Hinge4Type
    minspace: float
    maxspace: float

@dataclass(frozen=True)
class DoorDimensions:
    width: float
    depth: float  
    height: float
    
@dataclass(frozen=True)
class DoorSpacing:
    right: float
    left: float
    top: float
    bottom: float
    double: float

# core/domain/value_objects/door_constants.py
class DoorConstants:
    OPENSIDE_LEFT = "Left"
    OPENSIDE_RIGHT = "Right" 
    OPENSIDE_TOP = "Top"
    OPENSIDE_BOTTOM = "Bottom"
    OPENSIDE_NONE = "None"
```

### 2.2. Переработать сущности

```python
# core/domain/entities/door.py
from abc import ABC
from dataclasses import dataclass
from typing import List, Optional
from .value_objects import DoorDimensions, DoorSpacing, Hinge4Type

@dataclass
class Door(ABC):
    id: Optional[int] = None
    dimensions: Optional[DoorDimensions] = None
    spacing: Optional[DoorSpacing] = None
    hinge_type: Optional[Hinge4Type] = None
    opening_side: str = ""
    
    def calculate_real_dimensions(self) -> DoorDimensions:
        # Бизнес-логика расчета реальных размеров
        pass

# core/domain/entities/straight_door.py
@dataclass 
class StraightDoor(Door):
    door_type: str = "straight"
    is_assembly: bool = True
    auto_hinges: bool = True
    
    def validate_construction(self) -> bool:
        # Валидация специфичная для прямых дверей
        pass
```

### 2.3. Создать Domain Services

```python
# core/domain/services/door_calculator.py
class DoorCalculator:
    @staticmethod
    def calculate_hinge_positions(door: Door, facade_height: float) -> List[float]:
        # Логика расчета позиций петель
        pass
    
    @staticmethod
    def calculate_facade_dimensions(door: Door) -> Tuple[float, float]:
        # Логика расчета размеров фасада
        pass

# core/domain/services/door_validator.py  
class DoorValidator:
    @staticmethod
    def validate_hinge_clearance(hinge_type: Hinge4Type, spacing: float) -> bool:
        """Проверка допустимого свеса петли"""
        hinge_space = HingeSpaceType.get(hinge_type)
        return hinge_space.minspace <= spacing < hinge_space.maxspace
```

### 2.4. Use Cases (Application Layer)

```python
# core/application/use_cases/create_door.py
from typing import Protocol
from core.domain.entities import Door
from core.application.dto import DoorDTO

class DoorRepository(Protocol):
    def save(self, door: Door) -> None: ...
    def find_by_id(self, door_id: int) -> Door: ...

class CreateDoorUseCase:
    def __init__(self, door_repo: DoorRepository, validator: DoorValidator):
        self.door_repo = door_repo
        self.validator = validator
    
    def execute(self, door_dto: DoorDTO) -> Door:
        # Создание и валидация двери
        door = self._create_door_from_dto(door_dto)
        if self.validator.validate(door):
            self.door_repo.save(door)
            return door
        raise ValueError("Invalid door configuration")
```

### 2.5. Фабрики для сложных объектов

```python
# core/domain/factories/door_factory.py
class DoorFactory:
    @staticmethod
    def create_door(door_type: str, config: dict) -> Door:
        if door_type == "straight":
            return StraightDoorFactory.create(config)
        elif door_type == "standard":
            return StandardDoorFactory.create(config)
        elif door_type == "wingline":
            return WingLineFactory.create(config)
        raise ValueError(f"Unknown door type: {door_type}")

class StraightDoorFactory:
    @staticmethod
    def create(config: dict) -> StraightDoor:
        dimensions = DoorDimensions(
            width=config['w'], 
            depth=config['d'], 
            height=config['h']
        )
        return StraightDoor(dimensions=dimensions, **config)
```

## 3. Конкретные улучшения для вашего кода

### 3.1. Убрать God Class `StandartDoor`

Разделить на:

- `DoorSpecification` - параметры двери
- `DoorConstructor` - логика построения  
- `DoorValidator` - проверки
- `DoorRenderer` - отрисовка

### 3.2. Вынести зависимости от K3

```python
# infrastructure/persistence/k3_adapter.py
class K3DoorRepository:
    def __init__(self, k3_connection):
        self.k3 = k3_connection
    
    def save(self, door: Door) -> None:
        # Адаптер для сохранения в K3
        k3_data = self._convert_to_k3_format(door)
        self.k3.save_door(k3_data)
    
    def _convert_to_k3_format(self, door: Door) -> dict:
        # Конвертация из доменной модели в формат K3
        pass
```

### 3.3. Улучшить обработку ошибок

```python
# shared/exceptions/door_exceptions.py
class DoorConstructionError(Exception):
    pass

class InvalidHingeConfigurationError(DoorConstructionError):
    pass

class FacadeDimensionError(DoorConstructionError):
    pass
```

### 3.4. Добавить типизованные конфигурации

```python
# core/domain/value_objects/door_config.py
from pydantic import BaseModel

class DoorConfig(BaseModel):
    proto_id: int
    furn_type: str
    mater_id: int
    hinge_config: HingeConfig
    handle_config: HandleConfig
    
    class Config:
        frozen = True  # Immutable
```

## 4. Пример рефакторинга метода `Make()`

**Было:**

```python
def Make(self):
    # 100+ строк смешанной логики
    objects = []
    self._SetAddVars()
    self._VerifyExceptions()
    # ... сложная логика построения
```

**Стало:**

```python
# core/application/services/door_assembly_service.py
class DoorAssemblyService:
    def __init__(self, 
                 hinge_builder: HingeBuilder,
                 facade_builder: FacadeBuilder,
                 handle_builder: HandleBuilder):
        self.hinge_builder = hinge_builder
        self.facade_builder = facade_builder  
        self.handle_builder = handle_builder
    
    def assemble_door(self, door: Door) -> List[DoorComponent]:
        components = []
        
        # Фасад
        facade = self.facade_builder.build(door)
        components.append(facade)
        
        # Петли
        hinges = self.hinge_builder.build(door, facade)
        components.extend(hinges)
        
        # Ручки
        handles = self.handle_builder.build(door, facade)
        components.extend(handles)
        
        return components

# core/application/use_cases/create_door_assembly.py
class CreateDoorAssemblyUseCase:
    def execute(self, door_id: int) -> DoorAssembly:
        door = self.door_repo.find_by_id(door_id)
        components = self.assembly_service.assemble_door(door)
        return DoorAssembly(door=door, components=components)
```

## 5. Преимущества новой архитектуры

1. **Тестируемость** - Domain слой можно тестировать изолированно
2. **Замена K3** - Можно легко заменить K3 на другую БД
3. **Расширяемость** - Новые типы дверей через Factory
4. **Читаемость** - Каждый класс имеет одну ответственность
5. **Безопасность типов** - Pydantic модели для конфигурации

## 6. План миграции

1. **Фаза 1**: Вынести Value Objects и Entities
2. **Фаза 2**: Создать интерфейсы репозиториев  
3. **Фаза 3**: Реализовать Use Cases
4. **Фаза 4**: Адаптеры для K3
5. **Фаза 5**: Переписать существующие классы используя новую архитектуру

Такой подход позволит сохранить работоспособность системы во время рефакторинга и постепенно улучшать код.
