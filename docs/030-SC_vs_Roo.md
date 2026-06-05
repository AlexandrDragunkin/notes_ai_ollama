---
title: SourceCraft vs Roo Code — сравнение архитектуры
description: Детальное сравнение форматов, конфигурации и механизма Skills между SourceCraft Code Assistant и Roo Code
author: Александр Драгункин
date: 2025-06-05
---

# SourceCraft vs Roo Code — сравнение архитектуры

> **Источник:** Анализ на основе документации Roo Code (https://roocodeinc.github.io/Roo-Code/features/skills/) и практического опыта работы с SourceCraft Code Assistant в проекте CAD K3-Мебель.

---

## 1. Базовое утверждение

> SourceCraft наследует работу Roo Code, просто вместо папки `.roo/` у него `.codeassistant/`

**Вердикт: ВЕРНО, но с важными уточнениями.**

---

## 2. Сравнение конфигурационных файлов

| Компонент | Roo Code | SourceCraft | Совместимость |
|-----------|----------|-------------|:---:|
| **Кастомные режимы** | `.roomodes` | `.roomodes` | ✅ **ОБЩИЙ ФОРМАТ** |
| **MCP-конфиг** | `.roo/mcp.json` | `.codeassistant/mcp.json` | ❌ Разные пути |
| **Custom instructions** | `.roo/rules/` (директория .md) | `.codeassistant/instructions.md` (один файл) | ❌ Разный формат |
| **Игнорирование файлов** | `.rooignore` | `.codeassistantignore` | ❌ Разные имена |
| **AGENTS.md (mode-specific)** | `.roo/rules-{mode}/AGENTS.md` | `.codeassistant/rules-{mode}/AGENTS.md` | ❌ Разные пути |

### 2.1. `.roomodes` — общий формат

Это **единственный полностью совместимый формат**. Файл `.roomodes` в корне проекта будет работать и в Roo Code, и в SourceCraft без изменений.

```yaml
customModes:
  - slug: k3-designer
    name: K3-Designer
    description: Проектирование мебели в CAD K3
    roleDefinition: >-
      ...
    groups:
      - read
      - command
      - mcp
```

### 2.2. Custom instructions — разные расположения

**Roo Code:**
```
.roo/rules/                 # общие для всех режимов
.roo/rules-code/            # только для Code mode
.roo/rules-architect/       # только для Architect mode
```

**SourceCraft:**
```
.codeassistant/instructions.md           # общие (один файл)
.codeassistant/rules-code/AGENTS.md      # только для Code mode
.codeassistant/rules-architect/AGENTS.md # только для Architect mode
```

---

## 3. Skills — ключевое различие

### 3.1. Roo Code Skills

Roo Code использует **файловую систему** для пользовательских skills:

```
# Глобальные (доступны во всех проектах)
~/.roo/skills/{skill-name}/SKILL.md
~/.agents/skills/{skill-name}/SKILL.md     # cross-agent

# Проектные (только в текущем проекте)
.roo/skills/{skill-name}/SKILL.md
.agents/skills/{skill-name}/SKILL.md       # cross-agent

# Mode-specific
.roo/skills-code/{skill-name}/SKILL.md     # только в Code mode
.roo/skills-architect/{skill-name}/SKILL.md
```

**Структура SKILL.md:**
```markdown
---
name: pdf-processing
description: Extract text and tables from PDF files using Python libraries
---

# Instructions
...
```

**Механизм Progressive Disclosure (3 уровня):**
1. **Level 1 (Discovery)** — читается только frontmatter (name + description), индексируется
2. **Level 2 (Instructions)** — при совпадении запроса с description загружается полный SKILL.md
3. **Level 3 (Resources)** — по запросу из инструкций загружаются bundled файлы

**Ключевые возможности:**
- Пользователь может создавать свои skills
- Bundled resources (скрипты, шаблоны рядом с SKILL.md)
- Mode-specific директории (`skills-code/`, `skills-architect/`)
- Override priority (проектные переопределяют глобальные)
- Symlink support

### 3.2. SourceCraft Skills

SourceCraft использует **встроенную команду `skill()`**, загружающую skill из системного промпта:

```xml
<available_skills>
  <skill>
    <name>create-mcp-server</name>
    <description>Instructions for creating MCP servers...</description>
  </skill>
  <skill>
    <name>create-mode</name>
    <description>Instructions for creating custom modes...</description>
  </skill>
</available_skills>
```

**Ограничения:**
- Только предустановленные skills (`create-mcp-server`, `create-mode`)
- Нельзя создать пользовательский skill через файлы
- Нет bundled resources
- Нет cross-agent (`~/.agents/`)

### 3.3. Сравнительная таблица Skills

| Аспект | Roo Code | SourceCraft |
|--------|----------|-------------|
| **Хранение** | Файлы `.roo/skills/{name}/SKILL.md` | Встроенные в системный промпт |
| **Создание** | Создать файл SKILL.md с frontmatter | Использовать команду `skill()` |
| **Пользовательские** | ✅ Да | ❌ Нет |
| **Progressive disclosure** | ✅ 3 уровня | ✅ Через `skill()` |
| **Bundled resources** | ✅ Скрипты, шаблоны | ❌ Нет |
| **Mode-specific** | ✅ `skills-code/` и т.д. | ✅ Через `available_skills` |
| **Frontmatter** | name + description обязательны | Не требуется |
| **Cross-agent (.agents/)** | ✅ Да | ❌ Нет |
| **Symlink support** | ✅ Да | ❌ Нет |

---

## 4. Override Priority (приоритет переопределения)

### Roo Code (от высшего к низшему)

```
1. Проектный .roo mode-specific (.roo/skills-code/my-skill/)     — высший
2. Проектный .roo generic (.roo/skills/my-skill/)
3. Проектный .agents mode-specific (.agents/skills-code/my-skill/)
4. Проектный .agents generic (.agents/skills/my-skill/)
5. Глобальный .roo mode-specific (~/.roo/skills-code/my-skill/)
6. Глобальный .roo generic (~/.roo/skills/my-skill/)
7. Глобальный .agents mode-specific (~/.agents/skills-code/my-skill/)
8. Глобальный .agents generic (~/.agents/skills/my-skill/)        — низший
```

### SourceCraft

- Только встроенные skills из системного промпта
- Нет пользовательской системы приоритетов

---

## 5. Что создано в проекте ARLINE

Созданная система **НЕ является Skills в терминологии Roo Code**, а представляет собой:

| Компонент | Файл | Назначение |
|-----------|------|------------|
| **Кастомный режим** | [`.roomodes`](../../ARLINE/.roomodes) | Режим `k3-designer` с полной roleDefinition |
| **Промты** | [`odocs/prompts/`](../../ARLINE/odocs/prompts/) | 5 markdown-файлов с инструкциями |
| **Custom instructions** | [`.codeassistant/instructions.md`](../../ARLINE/.codeassistant/instructions.md) | Краткая справка для модели |

Если потребуется совместимость с Roo Code skills, нужно будет создать:
```
.roo/skills/k3-create-panel/SKILL.md
.roo/skills/k3-create-cabinet/SKILL.md
.roo/skills/k3-3d-primitives/SKILL.md
.roo/skills/k3-mprofile/SKILL.md
.roo/skills/k3-engine-basics/SKILL.md
```

Каждый `SKILL.md` с frontmatter:
```markdown
---
name: k3-create-panel
description: Create furniture panels with edge banding, slots, fixings and other treatments in CAD K3 using the engine module
---
```

---

## 6. Рекомендации

1. **`.roomodes`** — можно использовать как общий формат для обоих ассистентов
2. **Custom instructions** — нужно дублировать в `.roo/rules/` и `.codeassistant/instructions.md` при переключении
3. **Skills** — если нужна совместимость, создавать `.roo/skills/` структуру
4. **SourceCraft** — текущая структура (`.roomodes` + `odocs/prompts/` + `.codeassistant/instructions.md`) полностью покрывает потребности

---

## 7. Ссылки

- [Документация Roo Code Skills](https://roocodeinc.github.io/Roo-Code/features/skills/)
- [Файл .roomodes проекта ARLINE](../../ARLINE/.roomodes)
- [Промты K3 в ARLINE](../../ARLINE/odocs/prompts/)
- [Custom instructions ARLINE](../../ARLINE/.codeassistant/instructions.md)
- [029-MCP-Server-K3-CAD.md](029-MCP-Server-K3-CAD.md) — MCP сервер для CAD K3