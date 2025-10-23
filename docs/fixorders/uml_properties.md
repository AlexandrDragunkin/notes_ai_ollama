# Диаграмма свойств крепежа

```{uml}
@startuml
' UML диаграмма для fixordersutils/properties.py

package "fixordersutils" {
  class FixParsAdapter {
    -arr: k3.VarArray
    +fix_id
    +unit_id
    +put_from
    +mirrored
    +offset
    +length
    +contact_spot
    +__post_init__()
  }
  
  class _Params {
    -Length: k3.Var
    -hPanel: k3.Var
    -Poly: k3.Var
    -Side: k3.Var
    -IDHol: k3.VarArray
    -XHol: k3.VarArray
    -YHol: k3.VarArray
    -ZHol: k3.VarArray
    -FixPars: k3.VarArray
    -is_slot: bool
    -length_with_slot
    +__post_init__()
  }
  
  class Params {
    -params: _Params
  }
  
  class get_length_with_slot {
    +get_length_with_slot(panel: k3.Group, length: float, side: int)
  }
  
  class kvar {
    +kvar(v)
  }
  
  _Params --> FixParsAdapter
  _Params --> get_length_with_slot
}

note right of FixParsAdapter
  Класс-адаптер для расширеных параметров 
  крепежа в правиле с 9-ю парамерами
end note

note right of _Params
  Класс данных параметров закона 
  расстановки крепежа по 8-ми параметрам
end note

note right of Params
  Хранитель 9-ти параметров для 
  расстановки крепежа по макро
end note

@enduml