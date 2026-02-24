# VRML Параметризация сложного кухонного шкафа с множеством типов дверок 

Параметризация сложного кухонного шкафа с множеством типов дверок требует продуманной архитектуры. Вот несколько подходов:

## 1. Объектно-ориентированный подход с наследованием

```vrml
#VRML V2.0 utf8

# Базовый прототип дверцы
PROTO BaseDoor [
  field SFFloat width  0.5
  field SFFloat height 0.7
  field SFFloat thickness 0.018
  field SFString doorType "standard"
  field MFNode appearance NULL
]
{
  Script {
    url "javascript:
      function initialize() {
        generateGeometry();
      }
      
      function generateGeometry() {
        var coords = new MFVec3f();
        var indices = new MFInt32();
        
        // Базовая логика для прямоугольной дверцы
        if (doorType == 'standard') {
          createStandardDoor(coords, indices);
        } else if (doorType == 'glass') {
          createGlassDoor(coords, indices);
        }
        // ... другие типы
        
        Browser.setNodeField(this, 'geometry_coords', coords);
        Browser.setNodeField(this, 'geometry_indices', indices);
      }
    "
    
    eventOut MFVec3f geometry_coords
    eventOut MFInt32 geometry_indices
  }
  
  Shape {
    appearance IS appearance
    geometry DEF DoorGeometry IndexedFaceSet {
      coord Coordinate { point [ ] }
      coordIndex [ ]
    }
  }
}
```

## 2. Фабричный метод для создания дверок

```vrml
# Прототип фабрики дверок
PROTO DoorFactory [
  field SFInt32 doorStyle 1
  field SFFloat width  0.5
  field SFFloat height 0.7
  field MFNode customParams []
]
{
  Script {
    url "javascript:
      function initialize() {
        createDoor();
      }
      
      function createDoor() {
        var doorNode;
        
        switch (doorStyle) {
          case 1: // Стандартная прямоугольная
            doorNode = createStandardDoor();
            break;
          case 2: // Стеклянная с фрезеровкой
            doorNode = createGlassDoor();
            break;
          case 3: // Арочная
            doorNode = createArchedDoor();
            break;
          case 4: // С филёнками
            doorNode = createPaneledDoor();
            break;
          case 5: // Радиусная
            doorNode = createRadiusDoor();
            break;
          // ... и т.д. для 10+ типов
        }
        
        doors_changed[0] = doorNode;
      }
    "
    
    eventOut MFNode doors_changed
  }
}
```

## 3. Модульная система с внешними конфигурациями

### Главный файл шкафа:
```vrml
#VRML V2.0 utf8

# Конфигурация шкафа
DEF CabinetConfig Script {
  url "javascript:
    function getDoorConfig(doorIndex) {
      var configs = [
        { type: 'standard', width: 0.5, height: 0.7, style: 1 },
        { type: 'glass',    width: 0.4, height: 0.6, style: 2 },
        { type: 'arched',   width: 0.6, height: 0.8, style: 3 },
        { type: 'radius',   width: 0.3, height: 0.7, style: 5 },
        // ... 10+ конфигураций
      ];
      return configs[doorIndex];
    }
  "
  
  eventOut MFNode doorConfigs
}

# Загрузчик модулей дверок
DEF DoorLoader Script {
  url "javascript:
    function loadDoorModule(doorType) {
      var url = 'doors/' + doorType + '.wrl';
      Browser.createVrmlFromURL(url, this, 'vrmlLoaded');
    }
    
    function vrmlLoaded(nodes) {
      doors_changed = nodes;
    }
  "
  
  eventIn SFString loadDoorModule
  eventOut MFNode doors_changed
}
```

### Отдельный файл для каждого типа дверки:
**doors/standard.wrl:**
```vrml
#VRML V2.0 utf8
PROTO StandardDoor [
  field SFFloat width 0.5
  field SFFloat height 0.7
]
{
  # Специфичная геометрия стандартной дверцы
  Shape {
    geometry IndexedFaceSet {
      # ... параметризованная геометрия
    }
  }
}
```

## 4. Система с плагинами (наиболее гибкая)

```vrml
#VRML V2.0 utf8

# Регистратор плагинов дверок
DEF DoorPluginRegistry Script {
  url "javascript:
    var plugins = [];
    
    function registerPlugin(pluginNode) {
      plugins[plugins.length] = pluginNode;
    }
    
    function getPlugin(doorType) {
      for (var i = 0; i < plugins.length; i++) {
        if (plugins[i].pluginType == doorType) {
          return plugins[i];
        }
      }
      return null;
    }
  "
  
  eventIn SFNode registerPlugin
  eventOut MFNode plugins_changed
}

# Базовый плагин дверцы
PROTO DoorPlugin [
  field SFString pluginType "unknown"
  field SFString description ""
]
{
  Script {
    url "javascript:
      function initialize() {
        // Автоматическая регистрация
        Browser.setNodeField(DoorPluginRegistry, 'registerPlugin', this);
      }
      
      function createDoor(params) {
        // Абстрактный метод - реализуется в конкретных плагинах
      }
    "
  }
}

# Конкретный плагин для арочной дверцы
PROTO ArchedDoorPlugin [
  field SFString pluginType "arched"
  field SFString description "Арочная дверца с параметризуемой аркой"
]
{
  DoorPlugin {
    pluginType IS pluginType
    description IS description
  }
  
  Script {
    url "javascript:
      function createDoor(params) {
        var coords = new MFVec3f();
        var indices = new MFInt32();
        
        // Создание арочной геометрии на основе params
        createArchGeometry(coords, indices, params.width, params.height, params.archHeight);
        
        return Browser.createVrmlFromString('\
          Shape { \
            geometry IndexedFaceSet { \
              coord Coordinate { point ' + coords.toString() + ' } \
              coordIndex ' + indices.toString() + ' \
            } \
          }');
      }
    "
  }
}
```

## 5. JSON-подобная конфигурация в VRML

```vrml
# Конфигурация всего шкафа в виде структурированных данных
DEF CabinetConfiguration Script {
  url "javascript:
    var config = {
      overallSize: { width: 2.0, height: 2.4, depth: 0.6 },
      doors: [
        {
          type: 'standard',
          position: { x: 0, y: 0 },
          size: { width: 0.5, height: 0.7 },
          style: 'modern',
          material: 'wood_oak'
        },
        {
          type: 'glass',
          position: { x: 0.5, y: 0 },
          size: { width: 0.4, height: 0.6 },
          style: 'frosted',
          material: 'glass'
        },
        // ... 10+ конфигураций дверок
      ],
      shelves: [
        { position: 0.3, depth: 0.5 },
        { position: 1.2, depth: 0.5 }
      ]
    };
    
    function getConfig() {
      return config;
    }
  "
  
  eventOut MFNode configuration
}
```

## 6. Практический пример использования

```vrml
# Главный файл кухонного шкафа
VRML V2.0 utf8

# Импорт всех необходимых компонентов
EXTERNPROTO StandardDoor [field SFFloat width, field SFFloat height] "doors/standard.wrl"
EXTERNPROTO GlassDoor [field SFFloat width, field SFFloat height] "doors/glass.wrl"
# ... импорт остальных 10+ типов дверок

DEF Cabinet Script {
  url "javascript:
    function initialize() {
      createCabinet();
    }
    
    function createCabinet() {
      var doors = [];
      
      // Динамическое создание дверок разных типов
      doors[0] = StandardDoor { width: 0.5, height: 0.7 };
      doors[1] = GlassDoor { width: 0.4, height: 0.6 };
      doors[2] = ArchedDoor { width: 0.6, height: 0.8 };
      // ... и т.д.
      
      cabinetDoors_changed = doors;
    }
  "
  
  eventOut MFNode cabinetDoors_changed
}

Group {
  children [
    USE Cabinet.cabinetDoors_changed
    # Остальные компоненты шкафа
  ]
}
```

## Рекомендации по реализации:

1. **Используйте EXTERNPROTO** для модульности
2. **Разделите типы дверок по файлам** для удобства поддержки
3. **Создайте систему регистрации** для легкого добавления новых типов
4. **Используйте JSON-подобные структуры** для конфигурации
5. **Реализуйте фабричный метод** для создания объектов

Такой подход позволит легко добавлять новые типы дверок без изменения основной логики шкафа.