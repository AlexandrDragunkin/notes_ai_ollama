# ![](./pix/vrmllogo2.0.gif)

# The Virtual Reality Modeling Language

### ISO/IEC DIS 14772-1

This is the VRML/ISO Draft for International Standard
(DIS). This document will be submitted to ISO for a vote.

A [summary of changes](#Summary) is listed
below. See [ftp://ftp.megatek.com/pub/puk/Response.doc](ftp://ftp.megatek.com/pub/puk/Response.doc)
for a Word document that lists most of changes made (warning: this is
a very long document).

**Schedule**

```
22Feb.................Publish Editor's Draft of VRML97 DIS [done]
22Feb-12Mar...........3 week review period (send comments to rikk@best.com) [done]
14Mar.................Freeze the draft (DIS) [done]
6April................Publish DIS and release ISO ballot
~Jul..................ISO vote
~Sept.................Finalize document and publish International Standard (IS)

```

![](./pix/vrmlbar.gif)

Этот документ является переводом официальной и полной спецификацией **Virtual Reality Modeling Language**, (VRML), ISO/IEC DIS 14772.



| **Общее**                                            | **Положения**                                              | **Приложения**                                             |
| ---------------------------------------------------- | ---------------------------------------------------------- | ---------------------------------------------------------- |
| ![](./pix/cube.gif)[Предисловие](part1/foreword.md)  | ![](./pix/cube.gif)1 [Scope](part1/scope.md)               | ![](./pix/cube.gif)A [Grammar](part1/grammar.md)           |
| ![](./pix/cube.gif)[Введение](part1/introduction.md) | ![](./pix/cube.gif)2 [Ссылки](part1/references.md)         | ![](./pix/cube.gif)B [Examples](part1/examples.md)         |
|                                                      | ![](./pix/cube.gif)3 [Определения](part1/glossary.md)      | ![](./pix/cube.gif)C [Java](part1/java.md)                 |
|                                                      | ![](./pix/cube.gif)4 [Концепции](part1/concepts.md)        | ![](./pix/cube.gif)D [JavaScript](part1/javascript.md)     |
|                                                      | ![](./pix/cube.gif)5 [Nodes](part1/nodesRef.md)            | ![](./pix/cube.gif)E [Bibliography](part1/bibliography.md) |
|                                                      | ![](./pix/cube.gif)6 [Fields и Events](part1/fieldsRef.md) |                                                            |
|                                                      | ![](./pix/cube.gif)7 [Conformance](part1/conformance.md)   |                                                            |

**_Предисловие_**содержит общую информацию о процессе стандартизации.
для VRML. **_Введение_**описывает назначение и конструкцию.
критерии и характеристики VRML. Следующие положения определяют
технические характеристики VRML ISO/IEC DIS 14772-1:

1. **_Scope_** определяет проблемную область, к которой обращается VRML.

2. **_Ссылки_** перечисляет нормативные стандарты, на которые имеются ссылки.
    в спецификации.

3. **_Определения_** содержит глоссарий используемой терминологии.
    в спецификации.

4. **_Concepts_** описывает различные основы VRML.

5. **_Fields_**_**и Events**__ указывают
    типы данных, используемые узлами.

6. **_Nodes_**определяет синтаксис и семантику узлов VRML.

7. **_Conformance_**описывает минимальную поддержку.
требования к реализациям VRML.


В спецификацию включено несколько приложений:

1. **_Grammar_** представляет BNF(*Backus–Naur form*) для формата файла VRML.

2. **_Java_** описывает, как интегрируются сценарии VRML.
    с Явой.

3. **_JavaScript_** описывает, как интегрируются сценарии VRML.
    с JavaScript.

4. **_Examples_** включает в себя множество файлов примеров VRML.

5. **_Bibliography_** перечисляет информативные, нестандартные
    темы, указанные в спецификации.


![](./pix/vrmlbar.gif)

## Сводка изменений в спецификации с 4 августа

##### \[Note: Функциональные изменения выделены ЖИРНЫМ шрифтом..\]

#### Общие изменения


1. Исправлено большое количество опечаток и ошибок в документах.

2. Условные обозначения документов ISO были приняты повсеместно (в связи с тем, что
    это онлайн-документ HTML, многие из этих соглашений были проигнорированы.
    или интерпретировано).

3. Ссылка на Fields и Events перенесена в раздел 5, а узел
    Ссылка перенесена в пункт 6.

4. Приложение «Примеры» перенесено из Приложения B в Приложение D.

5. Последовательная нумерация всех разделов, подразделов, рисунков и таблиц.

6. Приложения (ранее называемые appendices) помечены как
    normative и informative.

7. Удалены неспецифические формулировки (например, советы пользователю).
    повсюду.

8. Все обсуждения соответствия перенесены в раздел 7.


#### Статья 1, Область применения

1. Этот раздел был переписан.


#### Пункт 2, Нормативные ссылки

1. Добавлено несколько новых ссылок.

2. Были уточнены некоторые ссылки.


#### Пункт 3, Определения

1. Добавлено много новых определений.

2. Были уточнены многие определения.


#### Раздел 4. Концепции.

1. Этот раздел был существенно переработан. Порядок
    был полностью изменен для лучшего представления. Добавлено несколько новых
    разделов и многие разделы были уточнены.

2. Добавлены две новые диаграммы.

3. **\+ и - являются недопустимыми первыми символами в именах DEF.**
4. _children_ ExposedFields зависят от порядка (ранее это
    был неопределенным).

5. Уточнены поля _ccw_, _solid_, И _creaseAngle_.

6. Прототипы были существенно переписаны для ясности. Разнообразие
    случаев было разъяснено (например, область действия имен DEF внутри
    PROTO).

7. Lighting model: fixed redundant ambient term, attentuation term, and
    spotlight eqns.

8. Разъяснено, как интерпретировать различные форматы файлов изображений как карты текстур.

9. Имена field/event в инструкции ROUTE не требуют _set\__
    и \_ _changed_ prefix/suffix. Если имя field/event (в
    операторе ROUTE) не найден (например, _foo_), браузер
    должен добавить префикс _set\__ ( _set\_foo_) и добавить
    суффикс _\_changed_ ( _foo\_changed_) к имени, пытаясь проверить, использует ли пользователь сокращенную запись.

#### Раздел 6, Ссылка на Nodes

01. Внесено множество мелких уточнений и исправлений.

02. Справочная информация: ограничьте поле _groundAngle_ от 0,0 до PI/2, а не PI (это была ошибка).

03. Цилиндр: исправлена ​​ошибка в расчете верха и низа (height/2).

04. CylinderSensor: исправлена ​​ошибка определения вращения диска.

05. ElevationGrid: исправлено несколько ошибок в уравнениях и уточнены все поля.

06. Выдавливание: исправлены ошибки и существенно переписано (включая новую фигуру).

07. **FontStyle: changed _family_ field from SFString to MFString**
    **to support orderered list of choices; similar to URL fields.**
08. IndexedFaceSet: fixed errors in the figure.

09. **MovieTexture: changed _duration\_changed_ eventOut from**
    **SFFloat to SFTime to be consistent with AudioClip.**
10. **NavigationInfo: new value for type field "ANY" (means**
    **browser can do whatever it wants. This was added because several**
    **browsers were doing this anyway (misinterpreting the spec) and thus we**
    **needed a mechanism to support the existing policy. This change implies**
    **that the other NavInfo _type_ s are now strictly enforced (see [NavigationInfo](part1/nodesRef.html#NavigationInfo)).**
11. NavigationInfo: Clarified that NavInfo type is restrictive (unless
     "ANY").

12. Sound: the text has been significantly rewritten for accuracy and
     clarity, and a figure has been added.

13. Sound: clarified that the _intensity_ field means audio volume
     or loudness, not the technical term _audio intensity_(which
     implies a perceptual logarithmic volume). Note that light _intensity_
     fields are also based on _perceptual intensity_ not _energy_
    _intensity_--we have tried to design the VRML parameters to be
     perceptual controls in order to make authoring easier. Some VRML
     browsers may need to perform the log conversion before invoking their
     respective audio library.

14. SpotLight: fixed errors in equations.

15. TextureTransform: fixed (reversed) errors in matrix equations (note:
     that some browsers may be implemented this literally and must change).

16. TextureTransform: allow _scale_ values from -infinity to
     +infinity.

17. Transform: clarified that negative scale is undefined.

18. Transform: fixed errors in matrix equations.

19. Transform: fixed typo that allowed zero _scale,_ ( _scale_
     must be > 0.0).

20. Viewpoint: clarified _jump_ = TRUE to mean an instantaneous
     hyperjump.

21. Viewpoint: clarified jump transitions for Anchor, and loadURL().

22. **Viewpoint: fixed error in FOV text; uses minimum angle for FOV,**
    **not maximum.**

#### Clause 7, Conformance and minimum support requirements

1. This clause was re-written.

2. **A variety of additions and changes were made to Annex 7.**
   **Conformance and minimum support requirements; see Table 7-1.**

#### Annex A, VRML Grammar definition

1. **\+ and - chararcters are illegal first characters for DEF names.**

#### Annex B, Java Scripting Reference

1. **A variety of clarifications, fixes, and improvements have been**
   **made. See [http://vs.sme.co.jp/~sugino/vrml2.0/Aug4\_to\_Dec2.txt](http://vs.sme.co.jp/~sugino/vrml2.0/Aug4_to_Dec2.txt)**
   **for some details.**

#### Annex C, JavaScript Scripting Reference

1. **A variety of clarifications, fixes, and improvements have been**
   **made. See [http://www.jch.com/~jch/vrml/jsdeltas.html](http://www.jch.com/~jch/vrml/jsdeltas.html)**
   **for partial list of changes.**

#### Annex D, Examples

1. Many errors have been corrected and links to valid .wrl files have been
    added.


#### Annex E, Bibliography

1. Several new references were added.


#### Annex F, Index

1. Nuked it.


![](./pix/vrmlbar.gif)

```
https://graphics.stanford.edu/courses/cs248-98-fall/Assignments/Assignment3/VRML2_Specification/spec/index.html

```

