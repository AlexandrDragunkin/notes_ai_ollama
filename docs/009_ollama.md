# Используем событийно ориентированную архитектуру

## Введение  

В консольном 3D‑CAD‑приложении, где модель мебели собирается через Python‑API (3.7), **событийно‑ориентированная архитектура (SOA)** позволяет:

1. **Разделить бизнес‑логику (расчёты, построение геометрии) и реакцию на изменения** (валидацию, кэш, вывод, логирование).  
2. **Уменьшить связность компонентов** – каждый модуль подписывается только на те события, которые его интересуют.  
3. **Упростить расширяемость** – добавить новый тип анализа (например, проверка на пересечения) – просто добавить слушатель.  
4. **Обеспечить “live‑update”** – изменение параметра (ширина, толщина) автоматически приводит к пересчёту всех зависимых частей без ручного вызова цепочки функций.

Ниже – пошаговое руководство, примеры кода и рекомендации, как построить такую архитектуру в Python 3.7.

---

## 1. Ключевые понятия

| Понятие | Что это? | Пример в контексте мебели |
|---------|----------|---------------------------|
| **Событие** | Маленький объект‑сообщение, описывающий *что* произошло. | `ParamChanged(part='door', name='thickness', new_value=2.0)` |
| **Издатель (Publisher)** | Любой объект, который *генерирует* событие. | Класс `Door` после изменения толщины. |
| **Подписчик (Subscriber / Listener)** | Функция/объект, который *реагирует* на событие. | Функция `update_assembly` пересчитывает габариты корпуса. |
| **Шина событий (Event Bus / Dispatcher)** | Центральный посредник, принимающий событие от издателя и доставляющий его всем подписчикам. | `EventBus.publish(event)`. |
| **Конвейер (Pipeline)** | Последовательность шагов, каждый из которых слушает/издаёт события, образуя цепочку обработки. | `ParamChanged → validate → recompute → render`. |

---

## 2. Выбор инструмента реализации шины

| Инструмент | Плюсы | Минусы | Как подключить в Python 3.7 |
|------------|-------|--------|-----------------------------|
| **Собственная простая шина** | Полный контроль, без зависимостей, легко кастомизировать. | Нужно писать boilerplate. | См. пример ниже. |
| **`blinker`** | Малый, популярный, поддерживает namespace, weak‑refs. | Требует отдельную библиотеку (`pip install blinker`). | `from blinker import signal` |
| **`pydispatcher`** | Ориентирована на класс‑ориентированный стиль. | Более старый, менее активный. | `pip install PyDispatcher` |
| **`asyncio` + `asyncio.Queue`** | Позволяет использовать асинхронные обработчики, полезно для длительных расчётов. | Требует `async`/`await`, чуть сложнее в синхронных скриптах. | Встроено в Python 3.7. |

Для большинства консольных скриптов достаточно **самой лёгкой шины** – 20‑30 строк кода. Ниже – её реализация и пример использования.

---

## 3. Минималистичная шина событий

```python
# event_bus.py
from collections import defaultdict
from typing import Callable, Any, Dict, List

class Event:
    """Базовый класс для всех событий."""
    def __init__(self, **payload):
        self.__dict__.update(payload)

class EventBus:
    """Простой диспетчер: подписка / публикация."""
    _subscribers: Dict[type, List[Callable[[Event], None]]] = defaultdict(list)

    @classmethod
    def subscribe(cls, event_type: type, handler: Callable[[Event], None]) -> None:
        """
        Подписаться на событие `event_type`.
        `handler` получает экземпляр события.
        """
        cls._subscribers[event_type].append(handler)

    @classmethod
    def publish(cls, event: Event) -> None:
        """Отправить событие всем подписчикам."""
        for handler in cls._subscribers[type(event)]:
            try:
                handler(event)
            except Exception as exc:
                # В продакшене логируем, а здесь просто печатаем.
                print(f"[EventBus] Ошибка в обработчике {handler}: {exc}")

# -------------------------------------------------
# Пример пользовательских событий
class ParamChanged(Event):
    """Параметр детали изменён."""
    # payload: part (str), name (str), new_value (Any)

class PartCreated(Event):
    """Создана новая часть модели."""
    # payload: part (BasePart)

class AssemblyUpdated(Event):
    """Обновлена сборка (общие габариты, коллизии)."""
    # payload: assembly (Assembly)
```

**Почему так?**

- **Тип‑ориентированная подписка** (по классу события) – простота и отсутствие строковых клавиш.  
- **Weak‑ref** не нужен, потому что шина хранит только ссылки на функции; при сборке мусора они исчезнут сами.  
- **Синхронный** – подходят для большинства расчётов, а при необходимости можно обернуть в `asyncio`.

---

## 4. Базовые сущности модели мебели

```python
# parts.py
from dataclasses import dataclass, field
from typing import Tuple, List
import math
from event_bus import EventBus, PartCreated, ParamChanged

# ------------------------------
# Класс общей детали
@dataclass
class BasePart:
    name: str
    width: float
    height: float
    depth: float
    material: str = "MDF"
    # Список геометрических объектов, возвращаемых CAD‑API
    cad_objects: List[Any] = field(default_factory=list)

    def __post_init__(self):
        # Сообщаем системе, что новая часть появилась
        EventBus.publish(PartCreated(part=self))

    # Универсальный метод для изменения параметра
    def set_param(self, param_name: str, value: float):
        if not hasattr(self, param_name):
            raise AttributeError(f"{self.__class__.__name__} не имеет атрибута {param_name}")
        setattr(self, param_name, value)
        EventBus.publish(ParamChanged(part=self.name, name=param_name, new_value=value))

# ------------------------------
# Пример частей: боковина, задняя стенка, дверца
class SidePanel(BasePart):
    def __init__(self, width, height, thickness):
        super().__init__(name="SidePanel",
                         width=width,
                         height=height,
                         depth=thickness)

class BackPanel(BasePart):
    def __init__(self, width, height, thickness):
        super().__init__(name="BackPanel",
                         width=width,
                         height=thickness,   # в этом случае depth = thickness
                         depth=height)

class Door(BasePart):
    def __init__(self, width, height, thickness):
        super().__init__(name="Door",
                         width=width,
                         height=height,
                         depth=thickness)

# -------------------------------------------------
# Коробка (Assembly) – собирает детали в единую модель
class Assembly:
    def __init__(self, name: str):
        self.name = name
        self.parts: List[BasePart] = []
        self.bounding_box: Tuple[float, float, float] = (0, 0, 0)  # w, h, d

        # Подписываемся на события, меняющие геометрию
        EventBus.subscribe(PartCreated, self._on_part_created)
        EventBus.subscribe(ParamChanged, self._on_param_changed)

    # ----------------------------------------------
    def _on_part_created(self, ev: PartCreated):
        part = ev.part
        self.parts.append(part)
        self._recalc_bounding_box()
        # При желании сразу отрисовать в CAD:
        self._create_cad_objects(part)

    def _on_param_changed(self, ev: ParamChanged):
        # Если параметр относится к любой из наших частей, пересчитываем.
        # Для простоты будем просто вызывать обновление коробки.
        self._recalc_bounding_box()
        # При необходимости – обновляем CAD‑объекты
        self._update_cad_for_part(ev.part)

    # ----------------------------------------------
    def _recalc_bounding_box(self):
        if not self.parts:
            self.bounding_box = (0, 0, 0)
            return
        w = max(p.width for p in self.parts)
        h = max(p.height for p in self.parts)
        d = max(p.depth for p in self.parts)
        self.bounding_box = (w, h, d)
        # Событие о полной сборке:
        EventBus.publish(AssemblyUpdated(assembly=self))

    # ----------------------------------------------
    # Обёртки над CAD‑API (здесь – псевдо‑функции)
    def _create_cad_objects(self, part: BasePart):
        """Создаёт CAD‑геометрию для новой детали."""
        # Пример: api.create_box(width, height, depth, material)
        cad_obj = api.create_box(part.width, part.height, part.depth,
                                 material=part.material,
                                 name=part.name)
        part.cad_objects.append(cad_obj)

    def _update_cad_for_part(self, part_name: str):
        """Найти часть и обновить её CAD‑объекты (пересоздать)."""
        part = next((p for p in self.parts if p.name == part_name), None)
        if part is None:
            return
        # Удаляем старые объекты из сцены
        for obj in part.cad_objects:
            api.delete(obj)
        part.cad_objects.clear()
        self._create_cad_objects(part)

# -------------------------------------------------
# Псевдо‑CAD‑модуль (для примера)
class FakeCAD:
    """Эмуляция простого API CAD‑системы."""
    def __init__(self):
        self.objects = []

    def create_box(self, w, h, d, material, name):
        obj = {"type": "box", "w": w, "h": h, "d": d,
               "material": material, "name": name}
        self.objects.append(obj)
        print(f"[CAD] Создан {name}: {w}×{h}×{d} ({material})")
        return obj

    def delete(self, obj):
        self.objects.remove(obj)
        print(f"[CAD] Удалён {obj['name']}")

# Создаём глобальный API‑объект, который будет использоваться в коде:
api = FakeCAD()
```

### Что делает код?

1. **`BasePart`** – базовый класс всех деталей. При инициализации генерирует событие `PartCreated`. При изменении параметров – `ParamChanged`.
2. **`Assembly`** – отвечает за сборку коробки. Подписывается на оба события, автоматически поддерживая **bounding‑box**, **CAD‑объекты** и публикует `AssemblyUpdated` при изменениях.
3. **`EventBus`** – центральный диспетчер, через который все компоненты общаются без прямой зависимости.
4. **`FakeCAD`** – имитация реального API (замените на ваш `cad_api`).

---

## 5. Пример полного сценария (сборка шкафа)

```python
# main.py
from parts import SidePanel, BackPanel, Door, Assembly, EventBus, AssemblyUpdated

# 1️⃣ Создаём сборку
cabinet = Assembly(name="Шкаф_1")

# 2️⃣ Подписываемся на событие полной сборки (можно вести лог, генерировать отчёт)
def on_assembly_updated(ev):
    asm = ev.assembly
    w, h, d = asm.bounding_box
    print(f"[INFO] Сборка «{asm.name}» обновлена: {w:.1f}×{h:.1f}×{d:.1f} (W×H×D)")

EventBus.subscribe(AssemblyUpdated, on_assembly_updated)

# 3️⃣ Создаём детали (каждая автоматически отправит PartCreated)
SidePanel(width=800, height=1800, thickness=18)   # левая боковина
SidePanel(width=800, height=1800, thickness=18)   # правая боковина
BackPanel(width=800, height=1800, thickness=5)    # задняя стенка
Door(width=600, height=1800, thickness=2)        # дверь

# 4️⃣ Меняем параметры «на лету» – событие ParamChanged автоматически запустит
#    пересчёт сборки и обновит CAD‑геометрию.
door = next(p for p in cabinet.parts if p.name == "Door")
door.set_param("width", 550)      # сужаем дверь
door.set_param("thickness", 3)    # меняем толщину

# 5️⃣ Добавляем очередную деталь (полка)
class Shelf(BasePart):
    def __init__(self, width, depth, thickness):
        super().__init__(name="Shelf",
                         width=width,
                         height=thickness,
                         depth=depth)

Shelf(width=800, height=30, thickness=18)  # полка 30 mm толщиной

# Финальный вывод уже сделан в слушателе on_assembly_updated.
```

**Вывод в консоль (пример):**
```
[CAD] Создан SidePanel: 800×1800×18 (MDF)
[CAD] Создан SidePanel: 800×1800×18 (MDF)
[CAD] Создан BackPanel: 800×5×1800 (MDF)
[CAD] Создан Door: 600×1800×2 (MDF)
[INFO] Сборка «Шкаф_1» обновлена: 800.0×1800.0×1800.0 (W×H×D)
[CAD] Удалён Door
[CAD] Создан Door: 550.0×1800.0×2 (MDF)
[INFO] Сборка «Шкаф_1» обновлена: 800.0×1800.0×1800.0 (W×H×D)
[CAD] Удалён Door
[CAD] Создан Door: 550.0×1800.0×3 (MDF)
[INFO] Сборка «Шкаф_1» обновлена: 800.0×1800.0×1800.0 (W×H×D)
[CAD] Создан Shelf: 800×30.0×18 (MDF)
[INFO] Сборка «Шкаф_1» обновлена: 800.0×1800.0×1800.0 (W×H×D)
```

---

## 6. Расширенные варианты событий

| Событие | Когда генерировать | Что может потребоваться в обработчике |
|---------|--------------------|----------------------------------------|
| `ParamChanged` | При изменении любого параметра детали (ширина, высота, материал). | Валидация диапазонов, пересчёт зависимостей, обновление визуализации. |
| `PartCreated` | После создания объекта `BasePart`. | Добавление в структуру сборки, автоматическое размещение в координатах. |
| `AssemblyUpdated` | Когда изменились габариты/структура всей сборки. | Запуск расчёта веса, экспорт BOM (Bill‑of‑Materials), проверка коллизий. |
| `CollisionDetected` | При проверке пересечений деталей. | Вывод предупреждений, автоматическое смещение/скейлинг. |
| `ExportReady` | После того, как модель готова к экспорту (STL, STEP). | Запуск функции `api.export(file_path)`. |
| `UserCommand` | При вводе команды в консоль (если у вас REPL‑интерфейс). | Привязывание к отдельным обработчикам команд. |

**Реализация пользовательской команды:**

```python
class UserCommand(Event):
    """Событие, генерируемое в REPL."""
    # payload: cmd (str), args (list)

def repl_loop():
    while True:
        line = input(">>> ").strip()
        if not line:
            continue
        parts = line.split()
        EventBus.publish(UserCommand(cmd=parts[0], args=parts[1:]))

# Пример обработчика:
def on_user_command(ev: UserCommand):
    if ev.cmd == "add_shelf":
        width = float(ev.args[0])
        depth = float(ev.args[1])
        thickness = float(ev.args[2])
        Shelf(width=width, depth=depth, thickness=thickness)

EventBus.subscribe(UserCommand, on_user_command)
```

---

## 7. Как связать события с реальным CAD‑API

### 7.1. Объекты‑обёртки

Если API предоставляет функции типа:

```python
obj = cad.create_box(w, h, d, material)
cad.set_position(obj, x, y, z)
cad.delete(obj)
```

Создайте **адаптер** в виде небольшого класса‑обёртки, который будет подписан на события и переводить их в вызовы API.

```python
class CADAdapter:
    def __init__(self, api):
        self.api = api
        EventBus.subscribe(PartCreated, self._on_part_created)
        EventBus.subscribe(ParamChanged, self._on_param_changed)
        EventBus.subscribe(AssemblyUpdated, self._on_assembly_updated)

    def _on_part_created(self, ev: PartCreated):
        part = ev.part
        obj = self.api.create_box(part.width, part.height, part.depth,
                                  material=part.material,
                                  name=part.name)
        part.cad_objects.append(obj)

    def _on_param_changed(self, ev: ParamChanged):
        part_name = ev.part
        part = next(p for p in Assembly.instances[0].parts if p.name == part_name)  # упрощение
        # Удаляем старый объект и создаём новый
        for obj in part.cad_objects:
            self.api.delete(obj)
        part.cad_objects.clear()
        self._on_part_created(PartCreated(part=part))

    def _on_assembly_updated(self, ev: AssemblyUpdated):
        # Например, обновляем «границы» рабочего пространства в CAD
        w, h, d = ev.assembly.bounding_box
        self.api.set_workspace_dimensions(w, h, d)

# Инициализация
cad_adapter = CADAdapter(api)   # `api` – ваш реальный объект CAD‑SDK
```

### 7.2. Асинхронные расчёты

Если расчёт геометрии тяжёлый (например, генерация фасадов с резьбой), используйте `asyncio` в обработчике:

```python
import asyncio

class AsyncCADAdapter(CADAdapter):
    async def _on_part_created(self, ev: PartCreated):
        part = ev.part
        # Долгий процесс – будем запускать в отдельном таске
        await asyncio.to_thread(self.api.create_complex_part, part)

# Запуск цикла
async def main():
    # Всё, что выше
    await asyncio.sleep(0)   # «прокачка» событий
    # дальше – пользователь вводит команды, генерируем события

asyncio.run(main())
```

---

## 8. Как избежать типичных проблем

| Проблема | Как предотвратить |
|----------|-------------------|
| **Циклические события** (изменение параметра -> событие -> обработчик меняет тот же параметр). | В обработчиках проверяйте `if new_value == old_value: return`. Можно добавить **флаг “processing”** в `BasePart`. |
| **Лишняя нагрузка** (слишком много пересчётов при пакетных изменениях). | Группируйте изменения в «транзакцию» – создайте событие `BatchUpdateStart` / `BatchUpdateEnd`, а обработчики откладывают действия до конца. |
| **Утечки памяти** (подписчики, не отписанные). | При уничтожении объектов вызывайте `EventBus.unsubscribe(handler)`. В примере использована простая подписка без отписки; в продакшене храните `weakref.WeakSet`. |
| **Трудно отлаживать поток событий**. | Добавьте логгер в `EventBus.publish`, выводящий `event.__class__.__name__` и источник. |
| **Зависимость от порядка подписчиков**. | Не полагайтесь на порядок: каждый обработчик должен быть **идемпотентным** (может выполниться несколько раз без эффекта). |

---

## 9. Чек‑лист для внедрения SOA в ваш CAD‑скрипт

1. **Определите доменные события**  
   - Параметр детали изменён (`ParamChanged`)  
   - Деталь создана (`PartCreated`)  
   - Сборка готова к экспорту (`ExportReady`)  
   - Пользователь ввёл команду (`UserCommand`)  

2. **Создайте централизованную шину** (`EventBus`).  
   - Убедитесь, что она импортируется везде, где нужен `publish`/`subscribe`.  

3. **Обёрните все публичные API‑вызовы CAD** в адаптер, подписанный на события.  

4. **Перепишите существующие «прямые» вызовы** (`door.width = 600`) на **методы, генерирующие событие** (`door.set_param('width', 600)`).  

5. **Добавьте слушатели**:
   - Валидация параметров.  
   - Пересчёт габаритов сборки.  
   - Обновление графики/экспорт.  

6. **Тестируйте**:  
   - Добавьте «мок»‑шину (записывающую список вызванных событий).  
   - Проверьте, что изменения параметров вызывают только нужные обработчики.  

7. **Оптимизируйте**:  
   - При пакетных изменениях используйте `BatchUpdateStart/End`.  
   - При длительных расчётах – `asyncio`/`ThreadPoolExecutor`.  

8. **Документируйте**:  
   - Схему событий (как в UML‑диаграмме).  
   - Соглашения: какие параметры могут менять какие части.  

---

## 10. Полный минимальный пример (один файл)

```python
# furniture_event_demo.py
import sys
from collections import defaultdict
from dataclasses import dataclass, field

# ---------- Шина событий ----------
class Event: pass

class EventBus:
    _subs = defaultdict(list)
    @classmethod
    def subscribe(cls, ev_type, handler):
        cls._subs[ev_type].append(handler)
    @classmethod
    def publish(cls, ev):
        for h in cls._subs[type(ev)]:
            h(ev)

# ---------- События ----------
class ParamChanged(Event):
    def __init__(self, part, name, new): self.part, self.name, self.new = part, name, new
class PartCreated(Event):
    def __init__(self, part): self.part = part
class AssemblyUpdated(Event):
    def __init__(self, assembly): self.assembly = assembly

# ---------- Модель ----------
@dataclass
class Part:
    name: str
    w: float; h: float; d: float
    def set(self, name, value):
        setattr(self, name, value)
        EventBus.publish(ParamChanged(self, name, value))
    def __post_init__(self):
        EventBus.publish(PartCreated(self))

# ---------- Сборка ----------
class Assembly:
    def __init__(self):
        self.parts = []
        EventBus.subscribe(PartCreated, self._add)
        EventBus.subscribe(ParamChanged, self._recalc)
    def _add(self, ev): self.parts.append(ev.part); self._recalc(ev)
    def _recalc(self, ev):
        if not self.parts: return
        w = max(p.w for p in self.parts)
        h = max(p.h for p in self.parts)
        d = max(p.d for p in self.parts)
        self.bb = (w, h, d)
        EventBus.publish(AssemblyUpdated(self))

# ---------- Пример использования ----------
def on_assembly_updated(ev):
    print(f"Обновлена сборка: BB={ev.assembly.bb}")

EventBus.subscribe(AssemblyUpdated, on_assembly_updated)

asm = Assembly()
Side = Part("Side", w=800, h=1800, d=18)
Back = Part("Back", w=800, h=5,   d=1800)
Door = Part("Door", w=600, h=1800, d=2)

Door.set('w', 550)               # генерирует ParamChanged → пересчёт

# Вывод:
# Обновлена сборка: BB=(800, 1800, 1800)
# Обновлена сборка: BB=(800, 1800, 1800)
# ...
```

> **В реальном проекте** вместо `print` используйте `logging`, а вместо `print` в CAD‑адаптере – вызовы вашего SDK (создание/удаление объектов, экспорт).

---

## 11. Заключение

- **События** позволяют избавиться от «жесткой» связь между расчётами и визуализацией.  
- **EventBus** в Python 3.7 легко реализовать в несколько строк; если нужна более «продвинутая» функциональность, подключайте `blinker` или `pydispatcher`.  
- **Планируйте события заранее** (что меняется, кто должен отреагировать) – это ускорит дальнейшее масштабирование проекта.  
- **Контролируйте количество пересчётов** – используйте батч‑обновления и асинхронность для тяжёлых операций.  
- **Тестируйте**: мок‑бит шины помогает убедиться, что каждый компонент реагирует только на нужные события и не вызывает лишних побочных эффектов.

Следуя этим принципам, ваш консольный 3D‑CAD‑скрипт будет **гибким, расширяемым и легко поддерживаемым**, а добавление новых функций (проверка коллизий, генерация BOM, экспорт в разные форматы) станет простым вопросом регистрации новых слушателей. 🚀