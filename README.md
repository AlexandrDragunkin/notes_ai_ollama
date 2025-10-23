# Notes AI & Ollama

Заметки на память о диалогах с AI и Ollama

## Описание

Этот репозиторий содержит мои заметки и размышления о работе с искусственным интеллектом, в частности с Ollama и различными языковыми моделями. Здесь собраны практические советы, инструкции и примеры кода, которые могут быть полезны при работе с AI-ассистентами.

## Структура проекта

- `docs/` - Документация и статьи в формате Markdown
- `docs/_build/` - Сгенерированный сайт (игнорируется в git)

## Локальная разработка

### Установка зависимостей

Для локальной разработки и сборки сайта вам потребуется:

1. Node.js (версия 18 или выше)
2. MyST CLI

Установите MyST CLI глобально:

```bash
npm install -g mystmd
```

### Сборка сайта

Для сборки статического сайта выполните:

```bash
cd docs
myst build --execute --html
```

Сгенерированный сайт будет доступен в директории `docs/_build/html`.

### Запуск локального сервера

Для запуска локального сервера разработки:

```bash
cd docs
myst start
```

Сайт будет доступен по адресу http://localhost:3000

## Публикация

Сайт автоматически публикуется на GitHub Pages при каждом пуше в ветку `main` с помощью GitHub Actions.

Текущая конфигурация GitHub Pages: https://alexandrdragunkin.github.io/notes_ai_ollama/

## Содержание

- [Ollama 1](docs/001_ollama_1.md) - Оптимизация работы с LLM в VS Code
- [Ollama 2](docs/002_ollama_2.md) - Настройка CodeLlama 7B в VS Code
- [Ollama 3 Cloud](docs/003_ollama_3Cloud.md) - Использование Ollama в облаке
- [Ollama 4 Что выбрать](docs/004_ollama_4Что выбрать.md) - Рекомендации по выбору моделей
- [Ollama 5 VPS и модели](docs/005_ollama_5_VPS_и_модели.md) - Выбор VPS для запуска моделей

## Лицензия

MIT

```{uml}
@startuml
' UML диаграмма для fixordersutils/builders.py

package "fixordersutils" {
  class Builders {
    +build_group_second_fixes(x:float, j:int, count_fixes:int, fix_details_id:iter, l_posit:list)
    +build_group_chet_fixes(x:float, j:int, count_fixes:int, fix_details_id:iter, l_posit:list)
    +build_group_notchet_fixes(x:float, j:int, count_fixes:int, fix_details_id:iter, l_posit:list)
    +build_group_notchet_fixes_nm(x:float, j:int, count_fixes:int, fix_details_id:iter, l_posit:list)
    +build_group_chet_fixes_one_last_fix(x:float, j:int, count_fixes:int, fix_details_id:iter, l_posit:list)
    +build_group_chet_fixes_one_last_fix_minus_reverce_x32(x:float, j:int, count_fixes:int, fix_details_id:iter, l_posit:list)
    +build_group_chet_fixes_one_last_fix_reverce_x32(x:float, j:int, count_fixes:int, fix_details_id:iter, l_posit:list)
    +build_group_chet_fixes_one_skant_fix_reverce_x32(x:float, j:int, count_fixes:int, fix_details_id:iter, l_posit:list)
    +build_group_one_fix_reverce_x32(x:float, j:int, count_fixes:int, fix_details_id:iter, l_posit:list)
    +build_fixes(x:float, j:int, count_fixes:int, fix_details_id:iter, l_posit:list)
  }
  
  class BuilderHelpers {
    +is_Skant(fixid)
    +get_group_number(j:int, fix_details_id:iter)
    +get_id_hol(det_id, fix_details_id)
    +get_x_hol(x, _sign, det_id, fix_details_id)
    +get_y_hol(det_id, fix_details_id)
    +get_z_hol(det_id, fix_details_id)
    +get_holePositionParams(x:float, j:int, det_id:tuple, fix_details_id:iter, _sign:int)
    +_get_sign(j, count_fixes)
  }
  
  class HolePosition {
    +IDHol
    +IDHold
    +XHol
    +YHol
    +ZHol
  }
  
  Builders --> HolePosition
  Builders --> BuilderHelpers
  BuilderHelpers --> HolePosition
}

note right of Builders
  Построители для различных типов крепежа
  Содержит функции для создания 
  различных конфигураций крепежа
end note

note right of BuilderHelpers
  Вспомогательные функции для построителей
  Помогают в создании параметров 
  для позиционирования крепежа
end note
@enduml

