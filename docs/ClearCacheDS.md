Для полной очистки кеша импортов в Python нужно выполнить несколько действий:

## 1. Основной способ - очистка `sys.modules`

```python
import sys
import importlib

def clear_import_cache():
    """Очищает кеш импортированных модулей"""
    # Создаем копию ключей, так как мы будем изменять словарь во время итерации
    modules_to_remove = []
    
    # Ищем модули для удаления (исключая встроенные и системные)
    for module_name in sys.modules:
        # Исключаем критически важные модули
        if module_name in sys.builtin_module_names:
            continue
        if module_name.startswith(('sys', 'builtins', '__main__', 'importlib')):
            continue
        modules_to_remove.append(module_name)
    
    # Удаляем модули из кеша
    for module_name in modules_to_remove:
        del sys.modules[module_name]
    
    # Сбрасываем кеш путей импорта
    importlib.invalidate_caches()

# Использование
clear_import_cache()
```

## 2. Более безопасный способ с фильтрацией

```python
import sys
import importlib

def safe_clear_import_cache():
    """Безопасная очистка кеша импортов"""
    
    # Модули, которые НЕЛЬЗЯ удалять
    protected_modules = {
        'sys', 'builtins', '__main__', 'importlib', 'os', 
        'types', 'codecs', '_frozen_importlib', '_imp', '_warnings'
    }
    
    # Дополнительно защищаем все встроенные модули
    protected_modules.update(sys.builtin_module_names)
    
    modules_to_remove = []
    for module_name in list(sys.modules.keys()):
        if (module_name not in protected_modules and 
            not module_name.startswith('_')):
            modules_to_remove.append(module_name)
    
    for module_name in modules_to_remove:
        try:
            del sys.modules[module_name]
        except KeyError:
            pass
    
    importlib.invalidate_caches()
    print(f"Удалено {len(modules_to_remove)} модулей из кеша")

safe_clear_import_cache()
```

## 3. Очистка байт-кода (`__pycache__`)

```python
import os
import shutil

def clear_pycache(directory='.'):
    """Удаляет все директории __pycache__"""
    for root, dirs, files in os.walk(directory):
        if '__pycache__' in dirs:
            pycache_path = os.path.join(root, '__pycache__')
            shutil.rmtree(pycache_path)
            print(f"Удалено: {pycache_path}")

# Использование
clear_pycache()
```

## 4. Комплексное решение

```python
import sys
import importlib
import os
import shutil

def full_import_cleanup():
    """Полная очистка кеша импортов и байт-кода"""
    
    # 1. Очистка sys.modules
    protected = set(sys.builtin_module_names) | {
        'sys', 'builtins', '__main__', 'importlib', 'os'
    }
    
    removed_count = 0
    for module_name in list(sys.modules.keys()):
        if module_name not in protected and not module_name.startswith('_'):
            del sys.modules[module_name]
            removed_count += 1
    
    # 2. Сброс кеша путей импорта
    importlib.invalidate_caches()
    
    # 3. Очистка байт-кода (опционально)
    for root, dirs, files in os.walk('.'):
        if '__pycache__' in dirs:
            shutil.rmtree(os.path.join(root, '__pycache__'))
    
    print(f"Очистка завершена. Удалено модулей: {removed_count}")

# Использование
full_import_cleanup()
```

## 5. Для конкретных модулей (перезагрузка)

```python
import importlib

def reload_modules(module_names):
    """Перезагружает указанные модули"""
    for name in module_names:
        if name in sys.modules:
            importlib.reload(sys.modules[name])

# Использование
reload_modules(['my_module', 'another_module'])
```

## Важные предупреждения:

1. **Не удаляйте системные модули** - это может сломать интерпретатор
2. **В интерактивных средах** (Jupyter, IPython) будьте осторожны
3. **Переменные могут стать невалидными** - объекты, созданные из удаленных модулей, могут перестать работать
4. **Лучшая альтернатива** - перезапуск интерпретатора

## Когда использовать:

- При динамической перезагрузке кода во время разработки
- В тестовых средах для изоляции тестов
- При разработке плагинов или систем горячей перезагрузки

Для большинства случаев достаточно перезапуска интерпретатора - это самый безопасный способ.