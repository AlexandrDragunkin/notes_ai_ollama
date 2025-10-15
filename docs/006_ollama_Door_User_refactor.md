# Архитектурный подход к рефакторингу большого пакета Door_user

## 1. Краткое резюме

| Что необходимо сделать | Лучший «модельный» подход | Почему это удобно |
|------------------------|---------------------------|------------------|
| **Изменять и структурировать огромный пакет** | **Clean Architecture + Domain‑Driven Design (DDD)**, где в центре – *ядро* (Domain), а внешние слои – *приложение* и *инфраструктура* | Позволяет «заморозить» бизнес‑логику и не трогать её при добавлении/смене графики, датабаз, сервисов и т.д. Разделение по слоям упрощает юнит‑тесты, CI‑проверки и в дальнейшем переход на микросервисы. |
| **Инструментальная поддержка рефакторинга** | `ruff` + `black`+`isort`, `mypy` (static typing), `pylint`/`flake8`, `rope` (pycharm), `radon` (complexity) + CI‑pipeline (GitHub Actions/GitLab CI) | Систематический «проверочный чек‑лист»: форматирование → статический анализ → тесты → CI. |
| **Генерация/авто‑рефакторинг** | **GPT‑4 Turbo / GitHub‑Copilot** (или local `llama.cpp` + fine‑tuned model) | Эксперт‑подсказки по созданию новых классов, паттернов, вынесении кода в новые пакеты, но человек‑проверка обязательна. |

Ниже — практический план, где и как применять модель и инструменты.

---

## 2. Архитектурный паттерн – Clean Architecture для «пакета»

```
myapp/
 ├─ core/          # Domain (entities, value‑objects, repository interfaces, services)
 │   ├─ entities/
 │   ├─ value_objects/
 │   ├─ services/
 │   └─ ports/          # Адаптеры (консоль, API, БД)
 ├─ app/          # Application (use‑cases, DTO, orchestration)
 ├─ infra/        # Concrete implementations (ORM, external APIs, file‑system)
 ├─ ui/           # CLI / API (FastAPI/Flask, command‑line)
 └─ tests/        # Тесты, упираясь во слои в порядке «domain → application → infra → ui»
```

### 2.1. Почему Clean Architecture стоит выбирать

| Преимущество | Как это помогает при рефакторинге |
|--------------|-----------------------------------|
|  **Стабильность ядра** | Бизнес‑логика (core) не зависит от внешних компонентов. Можно менять БД, UI, даже язык, не трогая ядро. |
|  **Поддержка интеграционных тестов** | Тесты в корне «domain» проверяют сущности изолированно. |
|  **Разделение ответственности** | В «infra» падает всё, что связано с файлами, сетью, сериализацией. |
|  **Проектная автономия** | При необходимости разнести пакеты на микросервисы можно выделить под‑проекты из «core» как библиотеки. |
|  **Превью кода** | Новые разработчики воспринимают структуру быстрее. |

> **Приклад:** если ваше приложение – «система расчёта дверей» (примерный код в вопросе), у вас уже есть `BaseDoor`, `StraightDoor` и т.д. Переносим все эти классы в `core/door`, интерфейсы репозиториев в `core/ports`, `StraightDoor` становится *сервисом* в `app/door`, а `ui`/`api` обращается к нему через `app`.

---

## 3. Система рефакторинга

| Шаг | Что делать | Как проверить |
|-----|------------|--------------|
| 1.  **Статический анализ** | Запустить `ruff .` – это объединяет `flake8`, `pyproject` и `isort`. | Вывести список предупреждений / ошибок. |
| 2.  **Форматирование** | Установить `black .` + `isort .` (или `black --check`). | Проверить, что стиль PEP‑8 соблюдён. |
| 3.  **Типы** | Включить `mypy --strict`. | Оставить только `type‑hints` (dataclasses, TypedDict). |
| 4.  **Проверка сложности** | `radon cc *.py` – подсчёт McCabe‑complexity. | Уменьшить >10, «extract‑method» для сложных функций. |
| 5.  **Тесты** | Писать `pytest` – как для «корня» (универсальных утилит), так и для «core». | Убедиться, что `pytest -q` покрывает 85 %+ кода. |
| 6.  **CI** | Создать GitHub Actions: *Lint → Test → Build*. | Автоматический `make`‑проверка. |
| 7.  **Проверьте зависимости** | `pipdeptree --freeze` – чтобы не было «circular imports». | Переработать импорт‑шаблоны, вынести их в `.well-known` / `__init__`. |
| 8.  **Отделите сервисы** | Вынесите каждую «сторону двери» в отдельный класс/модуль, например: `StraightDoor`, `StandartDoor`, `WingLine`. | Проверьте, что каждый модуль импортирует только `core`. |
| 9.  **Рефакторинг через LLM** | Составьте «список паттернов», которые нужно применить (например, «Extract Class» для `StandingDoorWingBuilder`). | Запустить GPT‑4 «Write Python class for {X}» и вручную проверить. |
|10. **Проверка после изменений** | Запустите `pytest`, `ruff`, `mypy`. | Отслеживайте regressions. |

> **Проблемы в примере**  
> *Независимая `priceinfo`*, *микс `globals()` и `__dict__`*, *глобальные переменные*, `class`‑объекты внутри функции, `typing` не используется.  
> Описание *стороны двери* (OPENSIDE, HNDMac) можно вынести в `core.value_objects`.  
> Всё множество импортов можно разбросать, так что `__init__`‑модули будут «мягкими».

---

## 4. Примеры паттернов, которые стоит применить

### 4.1. **Port & Adapter**
`core/ports/repository.py`  
```python
class DoorRepository(Protocol):
    def find_by_id(self, door_id: int) -> Door: ...
    def save(self, door: Door) -> None: ...
```
`infra/orm/knox.py`  
```python
class KnoxDoorRepository(DoorRepository):
    def find_by_id(self, door_id: int):
        # ORM‑запрос...
```

### 4.2. **Use‑Case (Application service)**
```python
class CreateStandardDoor:
    def __init__(self, repo: DoorRepository, logger: Logger):
        self.repo = repo
        self.logger = logger

    def __call__(self, dto: DoorDto) -> DoorDto:
        door = StandardDoor.from_dto(dto)
        self.repo.save(door)
        self.logger.info(f"Door {door.id} created")
        return door.to_dto()
```

### 4.3. **Factory for building façade**
```python
class FacadeFactory:
    def create(self, door: Door, facadetype: str) -> Facade:
        if facadetype == "wing":
            return WingFacade(door)
        else:
            return StandardFacade(door)
```

### 4.4. **Strategy Pattern для выбора механизма**
```python
class HingeStrategy(Protocol):
    def build(self, door: Door) -> List[Hinge]: ...

class StandardHingeStrategy(HingeStrategy): ...
class CornerHingeStrategy(HingeStrategy): ...

class HingeBuilder:
    def __init__(self, strategy: HingeStrategy):
        self.strategy = strategy

    def build(self, door: Door) -> List[Hinge]:
        return self.strategy.build(door)
```

---

## 5. Инструменты автоматизации

| Инструмент | Что делает | Как использовать |
|------------|------------|-----------------|
| **PyCharm / VSCode (плагин)** | Refactor → Extract Method / Class, Move | IDE‑рефакторинг «по‑живу». |
| **rope** (CLI) | Автоматизировать «Extract Class» | `rope extract-class --source mymodule.py --destination core/door/newclass.py` |
| **black & isort** | Форматирование кода | `black . && isort .` |
| **ruff** | Linter + formatter | `ruff check --fix .` |
| **mypy** | Точная проверка типов | `mypy --strict .` |
| **radon** | Сложность McCabe | `radon cc -a myapp/` |
| **pytest** | Unit‑тесты | `pytest -s -vv` |
| **pipenv / poetry** | Управление зависимостями | `poetry add`/`pipenv install` |
| **git diff-filter** | Проверка добавления новых импортов | `git blame` |
| **GitHub Actions** | CI‑pipeline | `.github/workflows/ci.yml` с lint, test, build |
| **Dependabot** | Авто‑обновление зависимостей | Включить в репозиторий |
| **GPT‑4 Turbo (openai)** | Генерация кода, refactoring prompts | Написать prompt «Вынеси из модулю X все классы Y» |

Пример CI‑пайплайна (GitHub Actions):

```yaml
name: ci

on:
  push:
  pull_request:

jobs:
  lint-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip
      - name: Install deps
        run: pip install -r requirements.txt
      - name: Ruff
        run: ruff check --fix .
      - name: Black
        run: black --check .
      - name: Isort
        run: isort --check .
      - name: Mypy
        run: mypy .
      - name: Tests
        run: pytest --cov=.

```

---

## 6. Как использовать LLM для рефакторинга

1. **Создайте «список изменений»**  
   * «Вынести все классы, работающие с фасадами, в пакет `core/door/`»  
   * «Перенести методы `GetHandle` и `SetHandle` в отдельный сервис `HandlingService`»  

2. **Промпт**  
   ```
   Given a Python file that contains many unrelated classes and functions,
   refactor it so that all door‑related entities live in a new package
   `core/door`.  Extract `StraightDoor`, `StandartDoor`, `WingLine` into that
   package.  Also extract the hinge builder into `core/door/hinge.py`.  Keep
   type hints, docstrings, and tests unchanged.  Return the patch.
   ```

3. **Проверьте ответ**  
   * Просмотрите сгенерированные изменения.  
   * Запустите `pytest`.  
   * Сравните diff с `ruff`.  

> **Важно** – LLM может делать предположения о структуре, но **всегда проверяйте**. Он не видит `git`‑статус и живые зависимости.

---

## 7. Практический чек‑лист для рефакторинга больших пакетов

| Задача | Как проверить | Что делать |
|--------|----------------|-----------|
| **Собрать текущий код** | `git ls-files '*.py'` | Сделайте «dump» до изменений. |
| **Проверка существующих тестов** | `pytest --maxfail=1` | Если тестов мало – напишите «угловые» cases. |
| **Найти «горячие» модули** | `radon cc` + `pylint` | Упориться на файлах > 30 LOC сложности. |
| **Изменить импорт‑порядок** | `isort` | Защитить от циклических зависимостей. |
| **Вынести бизнес‑логику в core** | **refactor → Extract Class** | Удалить лишние зависимости. |
| **Создать сервисы** | Написать `Service`-классы в `app/` | Отделить бизнес‑логики от инфраструктуры. |
| **Реализовать репозитории** | Создать `core/ports/repository.py`. | Инверсия зависимостей. |
| **Надоить типы** | `mypy --strict`. | Минимальный набор аннотаций. |
| **Организовать CI** | Создать GitHub Actions. | Проверка `ruff`, `pytest`, `mypy` | 
| **Документировать** | `pdoc`, `mkdocs`. | Сгенерировать API‑документацию. |
| **Проверка миграции данных** | Write migration scripts (alembic) | Убедиться, что данные корректно переносятся. |
| **Проверить совместимость** | Бэктесты: run old code + new code side‑by‑side. | Оставьте `legacy`‑подсистема, если нужно. |

---

## 8. Краткое «прошедшее» и где можно использовать AI

| Что | Как AI может помочь | Что делать |
|-----|---------------------|------------|
| Генерация классов по паттерну | GPT‑4 + «extract‑class» prompt | Создать шаблоны, потом поправить вручную |
| Перенос методa в отдельный сервис | prompt «Move method X to class Y» | Проверить вызовы и зависимости |
| Перевод legacy‑кода в `type‑annotated` | AI‑переименование и добавление аннотаций | Запуск `mypy` после |
| Авто‑управление зависимостями | GPT‑4 + `requirements.txt` | Сгенерировать `pyproject.toml` + `poetry.lock` |

> **Замечание:** AI работает быстро, но **не заменяет** человек‑тестер. Проводите `code review`, `tests`, `CI` после автогенерации.  

---

## 9. Итоги

1. **Разбейте проект на слои** (`core → app → infra → ui`).  
2. **Поставьте строгие правила**: `black`, `ruff`, `mypy`, unit‑тесты, CI.  
3. **Используйте паттерны**: Repository, Service, Factory, Strategy, Adapter.  
4. **Вынесите бизнес‑логику в `core`** и держите её полностью изолированной.  
5. **Автоматизируйте рутинные рефакторинги** через `rope`, `pycharm`, `LLM`.  
6. **Проверьте миграцию** при помощи `alembic` или ручных скриптов.  
7. **Документируйте изменения** – API, «чем отличается» от старой модели, где хранится каждая сущность.  

С таким подходом любые старые пакеты можно переструктурировать, улучшить читаемость, повысить покрытие тестами и обеспечить совместимость с будущими микросервисами — и всё это без «шумного» переписывания кода вручную. 🚀

## 10. Новая структура пакета

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

## 11. Конкретные преобразования классов

### 11.1. Вынести Value Objects

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

### 11.2. Переработать сущности

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

### 11.3. Создать Domain Services

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

### 11.4. Use Cases (Application Layer)

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

### 11.5. Фабрики для сложных объектов

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

## 12. Конкретные улучшения для вашего кода

### 12.1. Убрать God Class `StandartDoor`

Разделить на:

- `DoorSpecification` - параметры двери
- `DoorConstructor` - логика построения  
- `DoorValidator` - проверки
- `DoorRenderer` - отрисовка

### 12.2. Вынести зависимости от K3

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

### 12.3. Улучшить обработку ошибок

```python
# shared/exceptions/door_exceptions.py
class DoorConstructionError(Exception):
    pass

class InvalidHingeConfigurationError(DoorConstructionError):
    pass

class FacadeDimensionError(DoorConstructionError):
    pass
```

### 12.4. Добавить типизованные конфигурации

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

## 13. Пример рефакторинга метода `Make()`

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

## 14. Преимущества новой архитектуры

1. **Тестируемость** - Domain слой можно тестировать изолированно
2. **Замена K3** - Можно легко заменить K3 на другую БД
3. **Расширяемость** - Новые типы дверей через Factory
4. **Читаемость** - Каждый класс имеет одну ответственность
5. **Безопасность типов** - Pydantic модели для конфигурации

## 15. План миграции

1. **Фаза 1**: Вынести Value Objects и Entities
2. **Фаза 2**: Создать интерфейсы репозиториев  
3. **Фаза 3**: Реализовать Use Cases
4. **Фаза 4**: Адаптеры для K3
5. **Фаза 5**: Переписать существующие классы используя новую архитектуру

Такой подход позволит сохранить работоспособность системы во время рефакторинга и постепенно улучшать код.
