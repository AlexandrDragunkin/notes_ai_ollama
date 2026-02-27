# ![](../pix/vrmllogo2.0.gif)

# The Virtual Reality Modeling Language

# Annex B. Java Scripting Reference

### (normative)

### ISO/IEC DIS 14772-1

#### 4 April 1997

![](../pix/vrmlbar.gif)

## B.1 Introduction

This annex describes the Java classes and methods that allow Script
nodes (see " [6.40 Script](nodesRef.html#Script)")
to interact with associated scenes. See " [4.12 Scripting](concepts.html#4.12)" for a general
description of scripting languages in VRML. Note that Java is not
required by VRML, but any implementation of Java within VRML Script
nodes must conform with the requirements specified in this annex.

[B.1 Introduction](#B.1)

[B.2 Language](#B.2)

[B.3 Supported protocol in the Script node's _url_\
field](#B.3)

[B.3.1 _url_\
field](#B.3.1)

[B.3.2 File \
extension](#B.3.2)

[B.3.3 MIME \
type](#B.3.3)

[B.4 EventIn handling](#B.4)

[B.4.1 \
Description](#B.4.1)

[B.4.2 \
Parameter passing with Event objects](#B.4.2)

[B.4.3 \
processEvents() and processEvent() methods](#B.4.3)

[B.4.3.1 processEvents() method](#B.4.3.1)

[B.4.3.2 processEvent() method](#B.4.3.2)

[B.4.4 \
eventsProcessed() method](#B.4.4)

[B.4.5 \
shutdown() method](#B.4.5)

[B.4.6 \
initialize() method](#B.4.6)

[B.5 Accessing fields and events](#B.5)

[B.5.1 \
Accessing fields, eventIns and eventOuts of the Script](#B.5.1)

[B.5.2 \
Accessing fields, eventIns and eventOuts of other nodes](#B.5.2)

[B.5.3 \
Sending eventOuts or eventIns](#B.5.3)

[B.6 Exposed classes and methods for nodes and fields](#B.6)

[B.6.1 \
Introduction](#B.6.1)

[B.6.2 Field \
class and ConstField class](#B.6.2)

[B.6.3 Array \
handling](#B.6.3)

[B.6.3.1 Format](#B.6.3.1)

[B.6.3.2 Constructors and methods](#B.6.3.2)

[B.6.4 Node \
class](#B.6.4)

[B.6.5 \
Browser class](#B.6.5)

[B.6.6 \
User-defined classes and packages](#B.6.6)

[B.6.7 \
Standard Java packages](#B.6.7)

[B.7 Exceptions](#B.7)

[B.8 Examples](#B.8) [B.9 Class definitions](#B.9)

[B.9.1 Class \
hierarchy](#B.9.1)

[B.9.2 VRML \
packages](#B.9.2)

[B.9.2.1 vrml Package](#B.9.2.1)

[B.9.2.2 vrml.field Package](#B.9.2.2)

[B.9.2.3 vrml.node Package](#B.9.2.3) [B.10 \
Example of exception class](#B.10)

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)B.2  Language

[Java](http://java.sun.com)(tm) is an
object-oriented, platform-independent, multi-threaded, general-purpose
programming environment developed at [Sun \
Microsystems, Inc.](http://www.sun.com/) See [2.\[JAVA\]](references.html#[JAVA])
for a full description of the Java programming language. This annex
describes the Java bindings of VRML to the Script node.

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)B.3  Supported protocol in the script node's  _url_ field

### B.3.1  _url_ field

The _url_ field of the Script node contains the URL of a file
containing the Java bytecode, for example:

```
    Script {
      url "http://foo.co.jp/Example.class"
      eventIn SFBool start
    }

```

See " [2\. Normative References](references.html)"
for the definition of URL.

### B.3.2 File extension

The file extension for Java bytecode is **`.class`**.

### B.3.3 MIME type

The MIME type for Java bytecode is defined as follows:

```
    application/x-java

```

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)B.4  EventIn handling

### B.4.1 Description

Events to the Script node are passed to the corresponding Java method ( [processEvents()](#B.4.3.1) or [processEvent()](#B.4.3.2))
in the script. The script is specified in the _url_ field of the
Script node.

For a Java bytecode file specified in the _url_ field, the
following three conditions hold:

1. it shall contain the class definition whose name is exactly the same as
    the body of the file name

2. it shall be a subclass of the Script class (see " [B.9.2.3 vrml.node Package](#B.9.2.3)")

3. it shall be declared as a "public" class


For example, the following Script node has one eventIn whose name is _start_.

```
    Script {
      url "http://foo.co.jp/Example1.class"
      eventIn SFBool start
    }

```

This node points to the script file Example1.class. Its source
(Example1.java) looks like this:

```
    import vrml.*;
    import vrml.field.*;
    import vrml.node.*;

    public class Example1 extends Script {
        ...
        // This method is called when any event is received
        public void processEvent(Event e){
           // ... perform some operation ...
        }
    }

```

In the above example, when the _start_ eventIn is sent the
processEvent() method receives the eventIn and is executed.

### B.4.2 Parameter passing with Event objects

When a Script node receives an eventIn, a processEvent() or
processEvents() method in the file specified in the url field of the
Script node is called, which receives the eventIn as a Java object
(Event object, see " [B.4.3 processEvents() and processEvent() methods](#B.4.3)").

The Event object has three fields of information associated with it:
name, value, and timestamp, whose values are passed by the eventIn.
These can be retrieved using the corresponding method on the Event
object.

```
   public class Event implements Cloneable {
      public String getName();
      public ConstField getValue();
      public double getTimeStamp();
      // other methods ...
   }

```

Suppose that the eventIn type is SFXXX and eventIn name is eventInYYY,
then

1. getName() shall return the string "eventInYYY "

2. getValue() shall return ConstField containing the value of the eventIn

3. getTimeStamp() shall return a double (in seconds) containing the
    timestamp when the eventIn occurred (see " [4.11 Time](concepts.html#4.11)")


In the example below, the eventIn name is _start_ and the eventIn
value is cast to ConstSFBool. Also, the timestamp for the time when the
eventIn occurred is available as a double. These are passed as an Event
object to the processEvent() method:

```
    public void processEvent(Event e){
        if(e.getName().equals("start")){
              ConstSFBool v = (ConstSFBool)e.getValue();
              if(v.getValue()==true){
                   // ... perform some operation with e.getTimeStamp()...
              }
         }
    }

```

### B.4.3 processEvents() and processEvent() methods

#### B.4.3.1 processEvents() method

Authors can define a processEvents() method within a class that is
called when the script receives some set of events. The prototype of
the processEvents() method is **`public void processEvents(int**
**count, Event events[]);`**

_count_ indicates the number of events delivered. _events_ is
the array of events delivered. Its default behavior is to iterate over
each event, calling processEvent() on each one as follows:

```
    public void processEvents(int count, Event events[])
    {
        for (int i = 0; i < count; i++){
            processEvent(events[i]);
        }
    }

```

Although authors might change this operation by giving a user-defined
processEvents() method, in most cases, they only change the
processEvent() method and the eventsProcessed() method as described
below.

When multiple eventIns are routed from a single node to a single Script
node and eventIns which have the same timestamp are received,
processEvents() receives multiple events as an event array. Otherwise,
each incoming event invokes separate processEvents().

For example, the processEvents() method receives two events in the
following case, when the TouchSensor is activated:

```
    Transform {
      children [
        DEF TS TouchSensor {}
        Shape { geometry Cone {} }
      ]
    }
    DEF SC Script {
      url "Example.class"
      eventIn SFBool isActive
      eventIn SFTime touchTime
    }
    ROUTE TS.isActive  TO SC.isActive
    ROUTE TS.touchTime TO SC.touchTime

```

#### B.4.3.2 processEvent() method

Authors can define a processEvent() method within a class. The
prototype of the processEvent() is **`public void processEvent(Event event);`**

Its default behavior is no operation.

### B.4.4 eventsProcessed() method

Authors may define an eventsProcessed() method within a class that is
called after some set of events has been received. This allows Script
nodes that do not rely on the ordering of events received to generate
fewer events than an equivalent Script node that generates events
whenever events are received (see " [4.3.1 processEvents() method](#B.4.3.1)").
The eventsProcessed() method is called after every invocation of
processEvents().

The prototype of the eventsProcessed() method is **`public void**
**eventsProcessed();`**

Its default behavior is no operation.

### B.4.5 shutdown() method

Authors may define a shutdown() method within the Script class that is
called when the corresponding Script node is deleted or the world
containing the Script node is unloaded or replaced by another world (see " [4.12.3 Initialize() and shutdown()](concepts.html#4.12.3)").

The prototype of the shutdown() method is **`public void shutdown();`**

Its default behavior is no operation.

### B.4.6 initialize() method

Authors may define an initialize() method within the Script class that
is called **before** the browser presents the world to the
user and **before** any events are processed by any nodes in
the same VRML file as the Script node containing this script (see " [4.12.3 Initialize() and shutdown()](concepts.html#4.12.3)").
The various methods of the Script class such as getEventIn(),
getEventOut(), getExposedField(), and getField() are not guaranteed to
return correct values before the initialize() method has been executed.
The initialize() method is called once during the life of the Script
object.

The prototype of the initialize() method is **`public void**
**initialize();`**

Its default behavior is no operation. See [Example2.java](#Example2.class)
in B.5.1 for an example of a user-specified initialize() method.

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif) B.5  Accessing fields and events

### B.5.1 Accessing fields, eventIns and eventOuts of the  Script

The fields, eventIns, and eventOuts of a Script node are accessible
from its corresponding Script class. Each field defined in the Script
node is available to the Script class by using its name. Its value can
be read-from or written-into. This value is persistent across function
calls. EventOuts defined in the Script node can be read. EventIns
defined in the Script node can be written to.

Accessing the fields of the Script node can be done by using the
following three types of Script class methods:

1. **`Field getField(String fieldName)`**


    is the method to get the reference to the Script node's field whose
    name is _fieldName_. The return value can be converted to the
    appropriate subclass of the Field class, (see " [B.6.2 Field class and ConstField class](#B.6.2)").

2. **`Field getEventOut(String eventOutName)`**


    is the method to get the reference to the Script node's eventOut whose
    name is _eventOutName_. The return value can be converted to
    the appropriate subclass of the Field class, (see " [B.6.2 Field class and ConstField class](#B.6.2)").

3. **`Field getEventIn(String eventInName)`**


    is the method to get the reference to the Script node's eventIn whose
    name is _eventInName_. The return value can be converted to
    the appropriate subclass of the Field class, (see " [B.6.2 Field class and ConstField class](#B.6.2)").
    EventIn is a write-only field. When the getValue() method is invoked on
    a Field object obtained by the getEventIn() method, the return value is
    unspecified.


When the setValue(), set1Value(), addValue(), insertValue(), delete()
or clear() methods are invoked on a Field object obtained by the **getField()**
method, the new value is stored in the corresponding VRML node's field
(see also " [B.6.2 Field class and ConstField class](#B.6.2)"
and " [B.6.3 Array handling](#B.6.3)").
In the case of the set1Value(), addValue(), insertValue() or delete()
methods, essentially all elements of the VRML node's field are
retrieved, then the value specified as an argument is set, added,
inserted, deleted (as appropriate) to/from the elements, and then
stored as the elements in the corresponding VRML node's field. In the
case of the clear() method, all elements of a VRML node's field are
cleared (see [the definition of the clear() method](#clearMethod)).

When the setValue(), set1Value(), addValue(), insertValue(), delete()
or clear() methods are invoked on a Field object obtained by the **getEventOut()**
method, the call generates an eventOut in the VRML scene (see also
" [B.6.2 Field class and ConstField class](#B.6.2)"
and " [B.6.3 Array handling](#B.6.3)").
The effect of this eventOut is specified by the associated Route(s) in
the VRML scene. In the case of the set1Value(), addValue(),
insertValue() or delete() methods, essentially all elements of the VRML
node's eventOut are retrieved, then the value specified as an argument
is set, added, inserted or deleted (as appropriate) to/from the
elements, then stored as the elements in the corresponding VRML node's
eventOut, and then the eventOut is sent. In the case of the clear()
method, all elements of VRML node's eventOut are cleared and an
eventOut with zero elements is sent (see [the \
definition of the clear() method](#clearMethod)).

When the setValue() or clear() methods are invoked on a Field object
obtained by the **getEventIn()** method, the call generates an
eventIn to the Script node. When the set1Value(), addValue(),
insertValue() or delete() methods are invoked on a Field object
obtained by the getEventIn() method, the exception
(InvalidFieldChangeException) is thrown.

For example, the following Script node (Example2) defines an eventIn _start_,
a field _state_, and an eventOut _on_. The method
initialize() is invoked before any events are received, and the method
processEvent() is invoked when _start_ receives an event:

```
    Script {
      url      "Example2.class"
      eventIn  SFBool start
      field    SFBool state TRUE
      eventOut SFBool on
    }

```

Example2.java:

```
    // Example2 toggles a persistent field variable "state" in the VRML
    // Script node each time an eventIn "start" is received, then sets
    // eventOut "on" equal to the value of "state"
    import vrml.*;
    import vrml.field.*;
    import vrml.node.*;

    public class Example2 extends Script {
        private SFBool state; // field
        private SFBool on;    // eventOut

        public void initialize(){
            state = (SFBool) getField("state");
            on = (SFBool) getEventOut("on");
        }

        public void processEvent(Event e){
            if(state.getValue()==true){
                on.setValue(false); // set false to eventOut 'on'
                state.setValue(false);
            }
            else {
                on.setValue(true);  // set true to eventOut 'on'
                state.setValue(true);
            }
        }
    }

```

### B.5.2 Accessing fields, eventIns and eventOuts of other  nodes

If a script program has an access to a node, then any eventIn, eventOut
or exposedField of that node is accessible by using the getEventIn(),
getEventOut() or getExposedField() method defined in the node's class
(see " [B.6.4 Node class](#B.6.4)").

The typical way for a Script node to have an access to another VRML
node is to have an SFNode field which provides a reference to the other
node. The following Example3 shows how this is done:

```
    DEF SomeNode Transform {}
    Script {
      field SFNode node USE SomeNode # SomeNode is a Transform node
      eventIn SFVec3f pos            # new value to be inserted in
                                     #   SomeNode's translation field
      url "Example3.class"
    }

```

Example3.java:

```
    import vrml.*;
    import vrml.field.*;
    import vrml.node.*;

    public class Example3 extends Script {
        private SFNode node;   // field
        private SFVec3f trans; // translation field captured from remote
                               // Transform node

        public void initialize(){
            node = (SFNode) getField("node");
        }

        public void processEvent(Event e){
            // get the reference to the 'translation' field of the Transform node
            trans = (SFVec3f)((Node) node.getValue()).getExposedField("translation");
            // reset translation to value given in Event e, which is eventIn pos
            // in the VRML Script node.
            trans.setValue((ConstSFVec3f)e.getValue());
        }
    }

```

### B.5.3 Sending eventOuts or eventIns

Assume that the thread which executes processEvent() (or
processEvents()) is called 'main' thread and any other thread, except
for the 'main' thread, is called 'sub' thread. Sending
eventOuts/eventIns in the 'main' thread follows the model described in
" [4.10.3 Execution model](concepts.html#4.10.3)"
and sending eventOuts/eventIns in any 'sub' thread follows the model
described in " [4.12.6 Asynchronous scripts.](concepts.html#4.12.6)"

**In the 'main' thread:** Calling one of the setValue(), set1Value,
addValue(), insertValue(), clear() or delete() methods on an
eventOut/eventIn sends that event at that time. Calling the methods
multiple times during one execution of the thread still only sends one
event which corresponds to the first call of the method. All other
calls are ignored. The event is assigned the same timestamp as the
initial event which caused the main thread to execute.

**In the 'sub' thread:** Calling one of the setValue(), set1Value,
addValue(), insertValue(), clear() or delete() method on an
eventOut/eventIn sends that event at that time. Calling the methods
multiple times during one execution of the thread sends one event per
call of the method. The browser assigns the timestamp to the event.

**Note:** sending eventIns is ordinarily performed by the VRML
scene, not by Java scripts. Exceptions are possible as specified in
paragraph " [B.5.1 Accessing fields, eventIns and \
eventOuts of the Script](#B.5.1)."

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)B.6  Exposed classes and methods for nodes and fields

### B.6.1 Introduction

Java classes for VRML are defined in the packages: _vrml, vrml.node_
and _vrml.field_.

The Field class extends [Java's \
Object class](http://java.sun.com/books/Series/Tutorial/java/javaOO/objectclass.html) by default; thus, Field has the full functionality of
the Object class, including the [getClass()](http://java.sun.com/books/Series/Tutorial/java/javaOO/objectclass.html)
method. The rest of the package defines a "Const" read-only
class for each VRML [field type](fieldsRef.html), with a
getValue() method for each class; and another read/write class for each
VRML field type, with both getValue() and setValue() methods for each
class. A getValue() method converts a VRML type value into a Java type
value. A setValue() method converts a Java type value into a VRML type
value and sets it to the VRML field.

Some methods are listed as " [throws \
exception](http://java.sun.com/tutorial/java/exceptions/index.html)," meaning that errors are possible. It may be
necessary to write exception handlers (using Java's **catch()**
method) when those methods are used. Any method not listed as
"throws exception" is guaranteed to generate no exceptions.
Each method that throws an exception includes a prototype showing which
exception(s) can be thrown.

### B.6.2 Field class and ConstField class

All VRML data types have equivalent classes in Java. The Field class is
the root of all field types.

```
    public abstract class Field implements Cloneable {
        // methods
    }

```

This class has two types of subclasses: read-only classes and
read/write classes

1. **Read-only classes**


    These classes support the getValue() method. Some classes support
    additional convenience methods to get value(s) from the object.



   [ConstSFBool](#ConstSFBoolClass), [ConstSFColor](#ConstSFColorClass), [ConstMFColor](#ConstMFColorClass), [ConstSFFloat](#ConstSFFloatClass), [ConstMFFloat](#ConstMFFloatClass), [ConstSFImage](#ConstSFImageClass), [ConstSFInt32](#ConstSFInt32Class), [ConstMFInt32](#ConstMFInt32Class), [ConstSFNode](#ConstSFNodeClass), [ConstMFNode](#ConstMFNodeClass), [ConstSFRotation](#ConstSFRotationClass), [ConstMFRotation](#ConstMFRotationClass), [ConstSFString](#ConstSFStringClass), [ConstMFString](#ConstMFStringClass), [ConstSFVec2f](#ConstSFVec2fClass), [ConstMFVec2f](#ConstMFVec2fClass), [ConstSFVec3f](#ConstSFVec3fClass), [ConstMFVec3f](#ConstMFVec3fClass), [ConstSFTime](#ConstSFTimeClass), [ConstMFTime](#ConstMFTimeClass)

2. **Read/write classes**


    These classes support both getValue() and setValue() methods. If the
    class name is prefixed with **MF**(meaning that it is a multiple
    valued field class), the class also supports the set1Value(),
    addValue() and insertValue() methods. Some classes support additional
    convenience methods to get and set value(s) from the object.

   [SFBool](#SFBoolClass), [SFColor](#SFColorClass), [MFColor](#MFColorClass), [SFFloat](#SFFloatClass), [MFFloat](#MFFloatClass), [SFImage](#SFImageClass), [SFInt32](#SFInt32Class), [MFInt32](#MFInt32Class), [SFNode](#SFNodeClass), [MFNode](#MFNodeClass), [SFRotation](#SFRotationClass), [MFRotation](#MFRotationClass), [SFString](#SFStringClass), [MFString](#MFStringClass), [SFVec2f](#SFVec2fClass), [MFVec2f](#MFVec2fClass), [SFVec3f](#SFVec3fClass), [MFVec3f](#MFVec3fClass), [SFTime](#SFTimeClass), [MFTime](#MFTimeClass)


The VRML Field class and its subclasses have several methods to get and
set value(s): getSize(), getValue(), get1Value(), setValue(),
set1Value(), addValue(), insertValue(), clear(), delete() and
toString(). In these methods, getSize(), get1Value(), set1Value(),
addValue(), insertValue(), clear() and delete() are only available for
multiple value field classes (MF classes).

03. **`getSize()`**


     is the method to return the number of elements of each multiple value
     field class (MF class).

04. **`getValue()`**


     is the method to convert a VRML type value into a Java type value and
     return it.

05. **`get1Value(int index)`**


     is the method to convert a single VRML type value ( _index_-th
     element of an array) and return it as a single Java type value. The
     index of the first element is 0. Attempting to get an element beyond
     the length of the element array throws an exception
     (ArrayIndexOutOfBoundsException).

06. **`setValue(value)`**


     is the method to convert a Java type _value_ into a VRML type
     value and copy it to the target object.

07. **`set1Value(int index, value)`**


     is the method to convert from a Java type _value_ to a VRML
     type value and copy it to the _index_-th element of the target
     object. The index of the first element is 0. Attempting to set an
     element beyond the length of the element array throws an exception
     (ArrayIndexOutOfBoundsException).

08. **`addValue(value)`**


     is the method to convert from a Java type _value_ to a VRML
     type value and append it to the target object, thus adding an element.

09. **`insertValue(int index, value)`**


     is the method to convert from a Java type _value_ to a VRML
     type value and insert it as a new element at the _index_-th
     position, thus adding an element. The index of the first element is 0.
     Attempting to insert the element beyond the length of the element array
     throws an exception (ArrayIndexOutOfBoundsException).

10. **`clear()`**


     is the method to clear all elements in the target object so that it
     has no more elements in it.

11. **`delete(int index)`**


     is the method to delete the _index_-th element from the
     target object, thus decreasing the length of the element array by one.
     The index of the first element is 0. Attempting to delete the element
     beyond the length of the element array throws an exception
     (ArrayIndexOutOfBoundsException).

12. **`toString()`**


     is the method to return a String containing the VRML utf8 encoded
     value (or values) of the equivalent of the field. In the case of the
     SFNode(ConstSFNode) and MFNode (ConstMFNode),
     - **SFNode(ConstSFNode)**: the method returns the VRML utf8
       string that, if parsed as the value of an SFNode field, would produce
       this node. If the browser is unable to reproduce this node, the name of
       the node followed by the open brace and close brace shall be returned.
       Additional information may be included as one or more VRML comment
       strings.

    - **MFNode(ConstMFNode)**: the method returns the VRML utf8
       string that, if parsed as the value of a MFNode field, would produce
       this array of nodes. If the browser is unable to reproduce this node,
       the name of the nodes followed by the open brace and close brace shall
       be returned. Additional information may be included as one or more VRML
       comment strings

See also " [B.5.1 Accessing fields, eventIns and eventOuts of the Script](#B.5.1)",
" [B.6.3 Array handling](#B.6.3)", " [B.6.4 Node class](#B.6.4)" and " [B.9.2.1 vrml Package](#B.9.2.1)" for each
class' methods definition.

### B.6.3 Array handling

#### B.6.3.1 Format

Some constructors and other methods of the field classes take an array
as an argument.

1. **A single-dimensional**
   **array**

    Some constructors and other methods of the following classes take a
    single-dimensional array as an argument. The array is treated as
    follows:
   1. **ConstSFColor, ConstMFColor, SFColor and MFColor**

      ```
              float colors[]

      ```



       colors\[\] consists of a set of three float-values (representing red,
       green and blue).

   2. **ConstSFRotation, ConstMFRotation, SFRotation and MFRotation**

      ```
              float rotations[]

      ```



       rotations\[\] consists of a set of four float-values (representing axisX,
       axisY, axisZ and angle).

   3. **ConstSFVec2f, ConstMFVec2f, SFVec2f and MFVec2f**

      ```
              float vec2s[]

      ```



       vec2s\[\] consists of a set of two float-values (representing x and y).

   4. **ConstSFVec3f, ConstMFVec3f, SFVec3f and MFVec3f**

      ```
              float vec3s[]

      ```



       vec3s\[\] consists of a set of three float-values (representing x, y and
       z).

   5. **ConstSFImage and SFImage**

      ```
              byte pixels[]

      ```



       pixels\[\] consists of 2-dimensional pixel image. The ordering of the
       individual components for an individual pixel within the array of bytes
       will be as follows:


      ```
      # Comp.   byte[i]     byte[i + 1]    byte[i + 2]  byte[i + 3]
      -------   ----------  -----------    -----------  -----------
         1      intensity1  intensity2     intensity3    intensity4
         2      intensity1  alpha1         intensity2    alpha2
         3      red1        green1          blue1        red2
         4      red1        green1          blue1        alpha1

      ```



       The order of pixels in the array are to follow that defined in " [5.5 SFImage](fieldsRef.html#SFImage)". byte
       0 is pixel 0, starting from the bottom left corner.
2. **A single integer and a single-dimensional array**

    Some constructors and other methods take a single integer value (called _size_)
    and a single-dimensional array as arguments: for example, MFFloat(int
    size, float values\[\]). The _size_ parameter specifies the
    number of valid elements in the array - from 0-th element to ( _size_
    \- 1)-th element - all other values are ignored. This means that the
    method may be passed an array of length _size_ or larger. The
    same [rule for a \
    single-dimensional array](#RulesForSingleDimensionalArray) is applied to the valid elements.

3. **An array of arrays**

    Some constructors and other methods alternatively take an array of
    arrays as an argument. The array is treated as follows:
   1. **ConstMFColor and MFColor**

      ```
              float colors[][]

      ```



       colors\[\]\[\] consists of an array of sets of three float-values
       (representing red, green and blue).

   2. **ConstMFRotation and MFRotation**

      ```
              float rotations[][]

      ```



       rotations\[\]\[\] consists of an array of sets of four float-values
       (representing axisX, axisY, axisZ and angle).

   3. **ConstMFVec2f and MFVec2f**

      ```
              float vec2s[][]

      ```



       vec2s\[\]\[\] consists of an array of sets of two float-values
       (representing x and y).

   4. **ConstMFVec3f and MFVec3f**

      ```
              float vec3s[][]

      ```



       vec3s\[\]\[\] consists of an array of sets of three float-values
       (representing x, y and z).

#### B.6.3.2 Constructors and methods

The following describes how arrays are interpreted in detail for each
constructor and method.

Suppose _NA_ represents the number of elements in the array
specified as an argument of some constructors and other methods, and _NT_
represents the number of elements which the target object requires or
has. For example, if the target object is SFColor, it requires exactly
3 float values.

In the following description, suppose SF\* represents subclasses of
Field class, ConstSF\* represents subclasses of ConstField class, MF\*
represents subclasses of MField class and ConstMF\* represents
subclasses of ConstMField class.

1. **A**
   **single-dimensional array**

    In the following description, if the target object is




   - ConstSFColor and SFColor, _NT_ is exactly 3

   - ConstMFColor and MFColor, _NT_ is a multiple of 3, and _NA_
      is rounded down to a multiple of 3

   - ConstSFRotation and SFRotation, _NT_ is exactly 4

   - ConstMFRotation and MFRotation, _NT_ is a multiple of 4,
      and _NA_ is rounded down to a multiple of 4

   - ConstSFVec2f and SFVec2f, _NT_ is exactly 2

   - ConstMFVec2f and MFVec2f, _NT_ is a multiple of 2, and _NA_
      is rounded down to a multiple of 2

   - ConstSFVec3f and SFVec3f, _NT_ is exactly 3

   - ConstMFVec3f and MFVec3f, _NT_ is a multiple of 3, and _NA_
      is rounded down to a multiple of 3

   - ConstSFImage and SFImage, _NT_ is exactly _width_\* _height_\* _components_
      ( _width_, _height_ and number of _components_
      in the image, see " [5.5 \
      SFImage](fieldsRef.html#SFImage)")


   1. **For ConstSF\***
      **objects and SF\* objects**




       For all constructors and methods which take a single-dimensional array
       as an argument, the following rules are applied.



      _NA_ shall be larger than or equal to _NT_. If _NA_
       is larger than _NT_, the elements from the 0-th to the ( _NT_
       \- 1)-th element are used and remaining elements are ignored.
       Otherwise, an exception(ArrayIndexOutOfBoundsException) is thrown.


       For example, when the array is used as an argument of the setValue()
       for SFColor, the array shall contain at least 3 float values. If the
       array contains more than 3 float values, the first 3 values are used.

   2. **For ConstMF\* objects and MF\* objects**      - **For constructor.**

               The same [rule \
               for ConstSF\* and SF\* objects](#SingleArrayRuleForSFAndConstSFObject) is applied.


               For example, when the array is used as an argument of the constructor
               for MFColor, the array shall contain at least 3 float values. If the
               array contains 3N, 3N +1 or 3N + 2 float values, the first 3N values
               are used.

      - **For setValue() method.**

         If _NT_ is smaller than or equal to _NA_, _NT_
         is increased to _NA_ and then all elements of the
         array are copied into the target object. If _NT_ is
         larger than _NA_, _NT_ is decreased to _NA_
         and then all elements of the array are copied into the target object.

      - **For getValue() method.**

         If _NT_ is smaller than or equal to _NA_, all
         elements of the target object are copied into the first _NT_
         elemets of the array. If _NT_ is larger than _NA_,
         an exception (ArrayIndexOutOfBoundsException) is thrown.

      - F **or set1Value() method.**

         The target element (the _index_-th element) is treated
         as an SF\* object. So the same [rule for \
         ConstSF\* and SF\* objects](#SingleArrayRuleForSFAndConstSFObject) is applied.

      - **For get1Value() method.**

         The target element (the _index_-th element) is treated
         as an SF\* (or ConstSF\*) object. So the same [rule for \
         ConstSF\* and SF\* objects](#SingleArrayRuleForSFAndConstSFObject) is applied.

      - **For addValue() and insertValue() method.**

         The corresponding SF\* object is created using the argument, and then
         added to the target object or inserted into the target object.
2. **A single integer and a single-dimensional array**

    For all constructors and methods which take a single integer value
    (called _size_) and a single-dimensional array as arguments;
    for example, MFFloat(int size, float values\[\]), the following rule is
    applied.




    The _size_ parameter specifies the number of valid elements
    in the array from the 0-th element to the ( _size_ - 1)-th
    element; all other values are ignored. This means that the method may
    be passed an array of length _size_ or larger.


    The valid elements are copied to a new array and [the rules for a \
    single-dimensional array](#RulesForSingleDimensionalArrayInMethod) are applied to the new array for all
    methods.

3. **An array of arrays**

    This argument is used only for MF\* objects and ConstMF\* objects. In the
    following case, suppose _NA_ is the number of arrays (for
    example float f\[4\]\[3\], _NA_ is 4) specified as an argument of
    some constructors and other methods and _NT_ is the return
    value of [getSize()](#getSizeMethod) method of each
    object.
   - **For constructor.**


      The object which has _NA_ elements is created.

   - **For setValue() method.**


      If _NT_ is smaller than or equal to _NA_, _NT_
      is increased to _NA_ and then all elements of the array
      are copied into the target object. If _NT_ is larger than _NA_, _NT_
      is decreased to _NA_ and then all elements of the array
      are copied into the target object.

   - **For getValue() method.**


      If _NT_ is smaller than or equal to _NA_, all
      elements of the target object are copied into the array. If _NT_
      is larger than _NA_, an
      exception(ArrayIndexOutOfBoundsException) is thrown.

### B.6.4 Node class

The _[Node](#NodeClass)_ class has several methods:

1. **`String getType()`**


    is the method to return the type of the node.

2. **`ConstField getEventOut(String eventOutName)`**


    is the method to get the reference to the node's eventOut whose name
    is _eventOutName_. The return value can be converted to the
    appropriate subclass of the Field class, (see " [B.6.2 Field class and ConstField class](#B.6.2)").

3. **`Field getEventIn(String eventInName)`**


    is the method to get the reference to the node's eventIn whose name is _eventInName_.
    The return value can be converted to the appropriate subclass of the
    Field class, (see " [B.6.2 Field class and ConstField class](#B.6.2)").
    EventIn is a write-only field. When the getValue() method is invoked on
    a Field object obtained by the getEventIn() method, the return value is
    unspecified.

4. **`Field getExposedField(String exposedFieldName)`**


    is the method to get the reference to the node's exposedField whose
    name is _exposedFieldName_. The return value can be converted
    to the appropriate subclass of the Field class, (see " [B.6.2 Field class and ConstField class](#B.6.2)").

5. **`Browser getBrowser()`**


    is the method to get the browser object that this node is contained in
    (see " [B.6.5 Browser class](#B.6.5)").

6. **`String toString()`**


    is the same as the [toString()](#toStringMethod) method
    of SFNode (ConstSFNode).


When the setValue(), set1Value(), addValue(), insertValue(), delete()
or clear() methods are invoked on a Field object obtained by the **getExposedField()**
method, the call generates an eventOut in the VRML scene (see also
" [B.6.2 Field class and ConstField class](#B.6.2)"
and " [B.6.3 Array handling](#B.6.3)").
The effect of this eventOut is specified by the associated Route(s) in
the VRML scene. In the case of the set1Value(), addValue(),
insertValue() or delete() methods, essentially all elements of the VRML
node's exposedField are retrieved, then the value specified as an
argument is set, added, inserted or deleted (as appropriate) to/from
the elements, then stored as the elements in the corresponding VRML
node's exposedField, and then the eventOut is sent. In the case of the
clear() method, all elements of VRML node's exposedField are cleared
and an eventOut with zero elements is sent (see [the \
definition of the clear() method](#clearMethod)).

When the setValue() or clear() methods are invoked on a Field object
obtained by the **getEventIn()** method, the call generates an
eventIn in the VRML scene. When the set1Value(), addValue(),
insertValue() or delete() methods are invoked on the Field object, an
exception (InvalidFieldChangeException) is thrown.

### B.6.5 Browser class

This section lists the public Java interfaces to the [_Browser_](#BrowserClass) class, which allows scripts to
get and set browser information. For descriptions of the following
methods, see " [4.12.10 Browser Script Interface](concepts.html#4.12.10)."
Table B.1 lists the Browser class methods.

#### Table B.1: Browser class methods

### **Return value**

### **Method name**

`String`**`getName`** `()``String`**`getVersion`** `()``float`**`getCurrentSpeed`** `()``float`**`getCurrentFrameRate`** `()``String`**`getWorldURL`** `()``void`**`replaceWorld`** `(BaseNode[] nodes)``BaseNode[]`**`createVrmlFromString`** `(String vrmlSyntax) ``void`

**`createVrmlFromURL`** `(String[] url,
            BaseNode node,`

 `                  String
            event) ``void`

**`addRoute`** `(BaseNode fromNode, String
            fromEventOut, `

`         BaseNode
            toNode, String toEventIn)``void`

**`deleteRoute`** `(BaseNode fromNode,` ` String
            fromEventOut, `

`            BaseNode
            toNode, String toEventIn)``void`**`loadURL`** `(String[] url, String[]
            parameter)``void`**`setDescription`** `(String description)`

See " [B.9.2.1 vrml Package](#B.9.2.1)"
for each method's definition.

Table B.2 contains conversions from the types used in Browser class to
Java types.

#### Table B.2: VRML and Java types

**VRML type****Java type**

SFString

String

SFFloat

float

MFString

String\[\]

MFNode

BaseNode\[\]

When a relative URL is specified as an argument of the loadURL() and
createVrmlFromURL() method, the path is relative to the script file
containing these methods (see " [4.5.3 Relative URLs](concepts.html#4.5.2)").

### B.6.6 User-defined classes and packages

The Java classes defined by a user can be used in the Java program.
They are first searched from the directories specified in the CLASSPATH
environment variable and then the directory where the Java program is
placed.

If the Java class is in a package, this package is searched from the
directories specified in the CLASSPATH environment variable and then
the directory where the Java program is placed.

### B.6.7 Standard Java packages

Java programs have access to the full set of classes available in
java.\*. All parts of Java are required to work as "normal"
for Java. So all methods specified in this annex are required to be
thread-safe. The security model is browser specific.

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)B.7  Exceptions

Java methods may throw the following exceptions:

1. **InvalidFieldException**


    is thrown at the time getField() is executed and the field name is
    invalid.

2. **InvalidEventInException**


    is thrown at the time getEventIn() is executed and the eventIn name is
    invalid.

3. **InvalidEventOutException**


    is thrown at the time getEventOut() is executed and the eventOut name
    is invalid.

4. **InvalidExposedFieldException**


    is thrown at the time getExposedField() is executed and the
    exposedField name is invalid.

5. **InvalidVRMLSyntaxException**


    is thrown at the time createVrmlFromString(), createVrmlFromURL() or
    loadURL() is executed and the vrml syntax is invalid.

6. **InvalidRouteException**


    is thrown at the time addRoute() or deleteRoute() is executed and one
    or more of the arguments is invalid.

7. **InvalidFieldChangeException**


    may be thrown as a result of all sorts of illegal field changes, for
    example:
    1. Adding a node from one World as the child of a node in another World.

   2. Creating a circularity in a scene graph.

   3. Setting an invalid string on enumerated fields, such as the fogType
       field of the Fog node.

   4. Calling the set1Value(), addValue() or delete() on a Field object
       obtained by the getEventIn() method.
8. **ArrayIndexOutOfBoundsException**


    is generated at the time getValue(), set1Value(), insertValue() or
    delete() is executed and the index is out of bound (see " [B.6.2 Field class and ConstField class](#B.6.2)").
    This is the standard exception defined in the Java Array class.

9. **IllegalArgumentException**


    is generated at the time loadURL() or createVrmlFromURL() is executed
    and an error is occurred before retrieving the content of the url (see
    " [B.6.5 Browser class](#B.6.5)"). This is the
    standard exception defined in Java.


If exceptions are not caught by authors, a browser's behavior is
unspecified (see " [B.10 Example of exception class](#B.10)").

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)B.8  Examples

The following is an example of a [Script](nodesRef.html#Script) node which determines whether a given
color contains a lot of red. The Script node exposes a field, an
eventIn, and an eventOut:

```
    Script {
      field    SFColor currentColor 0 0 0
      eventIn  SFColor colorIn
      eventOut SFBool  isRed
      url      "Example4.class"
    }

```

The following is the source code for the Example4.java file that gets
called every time an eventIn is routed to the above Script node:

Example4.java:

```
import vrml.*;
import vrml.field.*;
import vrml.node.*;

public class Example4 extends Script {
    // Declare field(s)
    private SFColor currentColor;

    // Declare eventOut
    private SFBool isRed;

    // buffer for  SFColor.getValue().
    private float colorBuff[] = new float[3];

    public void initialize(){
       currentColor = (SFColor) getField("currentColor");
       isRed = (SFBool) getEventOut("isRed");
    }

    public void processEvent(Event e){
        // This method is called when a colorIn event is received
        currentColor.setValue((ConstSFColor)e.getValue());
    }

    public void eventsProcessed(){
        currentColor.getValue(colorBuff);
        if (colorBuff[0] >= 0.5) // if red is at or above 50%
            isRed.setValue(true);
    }
}

```

Details on when the methods defined in Example4.java are called may be
found in " [4.10.3 Execution Model](concepts.html#4.?)."

* * *

#### [Example5: createVrmlFromUrl()](\#B.6.5)

```
    Script {
      url "Example5.class"
      field   MFString target_url "foo.wrl"
      eventIn MFNode   nodesLoaded
      eventIn SFBool   trigger_event
    }

Example5.java:
  import vrml.*;
  import vrml.field.*;
  import vrml.node.*;

  public class Example5 extends Script {
      private MFString target_url; // field
      private Browser browser;

      public void initialize(){
          target_url = (MFString)getField("target_url");
          browser = this.getBrowser();
      }

      public void processEvent(Event e){
          if(e.getName().equals("trigger_event")){
               // do something and then fetch values
               String[] urls;
               urls = new String[target_url.getSize()];
               target_url.getValue(urls);
               browser.createVrmlFromURL(urls, this, "nodesLoaded");
          }
          if(e.getName().equals("nodesLoaded")){
              // do something
          }
      }
  }

```

* * *

#### [Example6: addRoute()](\#B.6.5)

```
    DEF TS TouchSensor {}
    Script {
      url     "Example6.class"
      field   SFNode fromNode USE TS
      eventIn SFBool clicked
      eventIn SFBool trigger_event
    }

```

Example6.java:

```
  import vrml.*;
  import vrml.field.*;
  import vrml.node.*;

  public class Example6 extends Script {

      private SFNode fromNode;
      private Browser browser;

      public void initialize(){
          fromNode = (SFNode) getField("fromNode");
          browser = this.getBrowser();
      }

      public void processEvent(Event e){
          if(e.getName().equals("trigger_event")){
              // do something and then add routing
              browser.addRoute(fromNode.getValue(), "isActive", this, "clicked");
          }
          if(e.getName().equals("clicked")){
              // do something
          }
      }
  }

```

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)B.9 Class  definitions

### B.9.1 Class hierarchy

The classes are divided into three packages: [vrml](#B.9.2.1), [vrml.field](#B.9.2.2) and [vrml.node](#B.9.2.3).

```
java.lang.Object
     |
     +- vrml.Event
     +- vrml.Browser
     +- vrml.Field
     |       +- vrml.field.SFBool
     |       +- vrml.field.SFColor
     |       +- vrml.field.SFFloat
     |       +- vrml.field.SFImage
     |       +- vrml.field.SFInt32
     |       +- vrml.field.SFNode
     |       +- vrml.field.SFRotation
     |       +- vrml.field.SFString
     |       +- vrml.field.SFTime
     |       +- vrml.field.SFVec2f
     |       +- vrml.field.SFVec3f
     |       |
     |       +- vrml.MField
     |       |       +- vrml.field.MFColor
     |       |       +- vrml.field.MFFloat
     |       |       +- vrml.field.MFInt32
     |       |       +- vrml.field.MFNode
     |       |       +- vrml.field.MFRotation
     |       |       +- vrml.field.MFString
     |       |       +- vrml.field.MFTime
     |       |       +- vrml.field.MFVec2f
     |       |       +- vrml.field.MFVec3f
     |       |
     |       +- vrml.ConstField
     |               +- vrml.field.ConstSFBool
     |               +- vrml.field.ConstSFColor
     |               +- vrml.field.ConstSFFloat
     |               +- vrml.field.ConstSFImage
     |               +- vrml.field.ConstSFInt32
     |               +- vrml.field.ConstSFNode
     |               +- vrml.field.ConstSFRotation
     |               +- vrml.field.ConstSFString
     |               +- vrml.field.ConstSFTime
     |               +- vrml.field.ConstSFVec2f
     |               +- vrml.field.ConstSFVec3f
     |               |
     |               +- vrml.ConstMField
     |                       +- vrml.field.ConstMFColor
     |                       +- vrml.field.ConstMFFloat
     |                       +- vrml.field.ConstMFInt32
     |                       +- vrml.field.ConstMFNode
     |                       +- vrml.field.ConstMFRotation
     |                       +- vrml.field.ConstMFString
     |                       +- vrml.field.ConstMFTime
     |                       +- vrml.field.ConstMFVec2f
     |                       +- vrml.field.ConstMFVec3f
     |
     +- vrml.BaseNode
             +- vrml.node.Node
             +- vrml.node.Script

java.lang.Exception
     |
     +- java.lang.RuntimeException
     |       +- java.lang.IllegalArgumentException
     |               +- vrml.InvalidEventInException
     |               +- vrml.InvalidEventOutException
     |               +- vrml.InvalidExposedFieldException
     |               +- vrml.InvalidFieldChangeException
     |               +- vrml.InvalidFieldException
     |               +- vrml.InvalidRouteException
     |
     +- vrml.InvalidVRMLSyntaxException

```

### B.9.2 VRML packages

#### B.9.2.1 vrml Package

```
package vrml;

public class Event implements Cloneable
{
   public String getName();
   public double getTimeStamp();
   public ConstField getValue();
   public Object clone();

   public String toString();   // This overrides a method in Object
}

public class Browser
{
   private Browser();
   public String toString();   // This overrides a method in Object

   // Browser interface
   public String getName();
   public String getVersion();

   public float getCurrentSpeed();

   public float getCurrentFrameRate();

   public String getWorldURL();
   public void replaceWorld(BaseNode[] nodes);

   public BaseNode[] createVrmlFromString(String vrmlSyntax)
     throws InvalidVRMLSyntaxException;

   public void createVrmlFromURL(String[] url, BaseNode node, String event)
     throws InvalidVRMLSyntaxException;

   public void addRoute(BaseNode fromNode, String fromEventOut,
                        BaseNode toNode, String toEventIn);

   public void deleteRoute(BaseNode fromNode, String fromEventOut,
                           BaseNode toNode, String toEventIn);

   public void loadURL(String[] url, String[] parameter)
     throws InvalidVRMLSyntaxException;

   public void setDescription(String description);
}

public abstract class Field implements Cloneable
{
   public Object clone();
}

public abstract class MField extends Field
{
   public abstract int getSize();
   public abstract void clear();
   public abstract void delete(int index);
}

public abstract class ConstField extends Field
{
}

public abstract class ConstMField extends ConstField
{
   public abstract int getSize();
}

//
// This is the general BaseNode class
//
public abstract class BaseNode
{
   // Returns the type of the node.  If the node is a prototype
   // it returns the name of the prototype.
   public String getType();

   // Get the Browser that this node is contained in.
   public Browser getBrowser();
}

```

#### B.9.2.2 vrml.field Package

```
package vrml.field;

public class SFBool extends Field
{
   public SFBool();
   public SFBool(boolean value);

   public boolean getValue();

   public void setValue(boolean b);
   public void setValue(ConstSFBool b);
   public void setValue(SFBool b);

   public String toString();   // This overrides a method in Object
}

public class SFColor extends Field
{
   public SFColor();
   public SFColor(float red, float green, float blue);

   public void getValue(float colors[]);
   public float getRed();
   public float getGreen();
   public float getBlue();

   public void setValue(float colors[]);
   public void setValue(float red, float green, float blue);
   public void setValue(ConstSFColor color);
   public void setValue(SFColor color);

   public String toString();   // This overrides a method in Object
}

public class SFFloat extends Field
{
   public SFFloat();
   public SFFloat(float f);

   public float getValue();

   public void setValue(float f);
   public void setValue(ConstSFFloat f);
   public void setValue(SFFloat f);

   public String toString();   // This overrides a method in Object
}

public class SFImage extends Field
{
   public SFImage();
   public SFImage(int width, int height, int components, byte pixels[]);

   public int getWidth();
   public int getHeight();
   public int getComponents();
   public void getPixels(byte pixels[]);

   public void setValue(int width, int height, int components,
                        byte pixels[]);
   public void setValue(ConstSFImage image);
   public void setValue(SFImage image);

   public String toString();   // This overrides a method in Object
}

public class SFInt32 extends Field
{
   public SFInt32();
   public SFInt32(int value);

   public int getValue();

   public void setValue(int i);
   public void setValue(ConstSFInt32 i);
   public void setValue(SFInt32 i);

   public String toString();   // This overrides a method in Object
}

public class SFNode extends Field
{
   public SFNode();
   public SFNode(BaseNode node);

   public BaseNode getValue();

   public void setValue(BaseNode node);
   public void setValue(ConstSFNode node);
   public void setValue(SFNode node);

   public String toString();   // This overrides a method in Object
}

public class SFRotation extends Field
{
   public SFRotation();
   public SFRotation(float axisX, float axisY, float axisZ, float angle);

   public void getValue(float rotations[]);

   public void setValue(float rotations[]);
   public void setValue(float axisX, float axisY, float axisZ,
                        float angle);
   public void setValue(ConstSFRotation rotation);
   public void setValue(SFRotation rotation);

   public String toString();   // This overrides a method in Object
}

public class SFString extends Field
{
   public SFString();
   public SFString(String s);

   public String getValue();

   public void setValue(String s);
   public void setValue(ConstSFString s);
   public void setValue(SFString s);

   public String toString();   // This overrides a method in Object
}

public class SFTime extends Field
{
   public SFTime();
   public SFTime(double time);

   public double getValue();

   public void setValue(double time);
   public void setValue(ConstSFTime time);
   public void setValue(SFTime time);

   public String toString();   // This overrides a method in Object
}

public class SFVec2f extends Field
{
   public SFVec2f();
   public SFVec2f(float x, float y);

   public void getValue(float vec2s[]);
   public float getX();
   public float getY();

   public void setValue(float vec2s[]);
   public void setValue(float x, float y);
   public void setValue(ConstSFVec2f vec);
   public void setValue(SFVec2f vec);

   public String toString();   // This overrides a method in Object
}

public class SFVec3f extends Field
{
   public SFVec3f();
   public SFVec3f(float x, float y, float z);

   public void getValue(float vec3s[]);
   public float getX();
   public float getY();
   public float getZ();

   public void setValue(float vec3s[]);
   public void setValue(float x, float y, float z);
   public void setValue(ConstSFVec3f vec);
   public void setValue(SFVec3f vec);

   public String toString();   // This overrides a method in Object
}

public class MFColor extends MField
{
   public MFColor();
   public MFColor(float colors[][]);
   public MFColor(float colors[]);
   public MFColor(int size, float colors[]);

   public void getValue(float colors[][]);
   public void getValue(float colors[]);

   public void get1Value(int index, float colors[]);
   public void get1Value(int index, SFColor color);

   public void setValue(float colors[][]);
   public void setValue(float colors[]);
   public void setValue(int size, float colors[]);
   /****************************************************
    color[0] ... color[size - 1] are used as color data
    in the way that color[0], color[1], and color[2]
    represent the first color. The number of colors
    is defined as "size / 3".
    ***************************************************/
   public void setValue(MFColor colors);
   public void setValue(ConstMFColor colors);

   public void set1Value(int index, ConstSFColor color);
   public void set1Value(int index, SFColor color);
   public void set1Value(int index, float red, float green, float blue);

   public void addValue(ConstSFColor color);
   public void addValue(SFColor color);
   public void addValue(float red, float green, float blue);

   public void insertValue(int index, ConstSFColor color);
   public void insertValue(int index, SFColor color);
   public void insertValue(int index, float red, float green, float blue);

   public String toString();   // This overrides a method in Object
}

public class MFFloat extends MField
{
   public MFFloat();
   public MFFloat(int size, float values[]);
   public MFFloat(float values[]);

   public void getValue(float values[]);

   public float get1Value(int index);

   public void setValue(float values[]);
   public void setValue(int size, float values[]);
   public void setValue(MFFloat value);
   public void setValue(ConstMFFloat value);

   public void set1Value(int index, float f);
   public void set1Value(int index, ConstSFFloat f);
   public void set1Value(int index, SFFloat f);

   public void addValue(float f);
   public void addValue(ConstSFFloat f);
   public void addValue(SFFloat f);

   public void insertValue(int index, float f);
   public void insertValue(int index, ConstSFFloat f);
   public void insertValue(int index, SFFloat f);

   public String toString();   // This overrides a method in Object
}

public class MFInt32 extends MField
{
   public MFInt32();
   public MFInt32(int size, int values[]);
   public MFInt32(int values[]);

   public void getValue(int values[]);

   public int get1Value(int index);

   public void setValue(int values[]);
   public void setValue(int size, int values[]);
   public void setValue(MFInt32 value);
   public void setValue(ConstMFInt32 value);

   public void set1Value(int index, int i);
   public void set1Value(int index, ConstSFInt32 i);
   public void set1Value(int index, SFInt32 i);

   public void addValue(int i);
   public void addValue(ConstSFInt32 i);
   public void addValue(SFInt32 i);

   public void insertValue(int index, int i);
   public void insertValue(int index, ConstSFInt32 i);
   public void insertValue(int index, SFInt32 i);

   public String toString();   // This overrides a method in Object
}

public class MFNode extends MField
{
   public MFNode();
   public MFNode(int size, BaseNode node[]);
   public MFNode(BaseNode node[]);

   public void getValue(BaseNode node[]);

   public BaseNode get1Value(int index);

   public void setValue(BaseNode node[]);
   public void setValue(int size, BaseNode node[]);
   public void setValue(MFNode node);
   public void setValue(ConstMFNode node);

   public void set1Value(int index, BaseNode node);
   public void set1Value(int index, ConstSFNode node);
   public void set1Value(int index, SFNode node);

   public void addValue(BaseNode node);
   public void addValue(ConstSFNode node);
   public void addValue(SFNode node);

   public void insertValue(int index, BaseNode node);
   public void insertValue(int index, ConstSFNode node);
   public void insertValue(int index, SFNode node);

   public String toString();   // This overrides a method in Object
}

public class MFRotation extends MField
{
   public MFRotation();
   public MFRotation(float rotations[][]);
   public MFRotation(float rotations[]);
   public MFRotation(int size, float rotations[]);

   public void getValue(float rotations[][]);
   public void getValue(float rotations[]);

   public void get1Value(int index, float rotations[]);
   public void get1Value(int index, SFRotation rotation);

   public void setValue(float rotations[][]);
   public void setValue(float rotations[]);
   public void setValue(int size, float rotations[]);
   public void setValue(MFRotation rotations);
   public void setValue(ConstMFRotation rotations);

   public void set1Value(int index, ConstSFRotation rotation);
   public void set1Value(int index, SFRotation rotation);
   public void set1Value(int index, float axisX, float axisY, float axisZ, float angle);

   public void addValue(ConstSFRotation rotation);
   public void addValue(SFRotation rotation);
   public void addValue(float axisX, float axisY, float axisZ, float angle);

   public void insertValue(int index, ConstSFRotation rotation);
   public void insertValue(int index, SFRotation rotation);
   public void insertValue(int index, float axisX, float axisY, float axisZ, float angle);

   public String toString();   // This overrides a method in Object
}

public class MFString extends MField
{
   public MFString();
   public MFString(int size, String s[]);
   public MFString(String s[]);

   public void getValue(String s[]);

   public String get1Value(int index);

   public void setValue(String s[]);
   public void setValue(int size, String s[]);
   public void setValue(MFString s);
   public void setValue(ConstMFString s);

   public void set1Value(int index, String s);
   public void set1Value(int index, ConstSFString s);
   public void set1Value(int index, SFString s);

   public void addValue(String s);
   public void addValue(ConstSFString s);
   public void addValue(SFString s);

   public void insertValue(int index, String s);
   public void insertValue(int index, ConstSFString s);
   public void insertValue(int index, SFString s);

   public String toString();   // This overrides a method in Object
}

public class MFTime extends MField
{
   public MFTime();
   public MFTime(int size, double times[]);
   public MFTime(double times[]);

   public void getValue(double times[]);

   public double get1Value(int index);

   public void setValue(double times[]);
   public void setValue(int size, double times[]);
   public void setValue(MFTime times);
   public void setValue(ConstMFTime times);

   public void set1Value(int index, double time);
   public void set1Value(int index, ConstSFTime time);
   public void set1Value(int index, SFTime time);

   public void addValue(double time);
   public void addValue(ConstSFTime time);
   public void addValue(SFTime time);

   public void insertValue(int index, double time);
   public void insertValue(int index, ConstSFTime time);
   public void insertValue(int index, SFTime time);

   public String toString();   // This overrides a method in Object
}

public class MFVec2f extends MField
{
   public MFVec2f();
   public MFVec2f(float vec2s[][]);
   public MFVec2f(float vec2s[]);
   public MFVec2f(int size, float vec2s[]);

   public void getValue(float vec2s[][]);
   public void getValue(float vec2s[]);

   public void get1Value(int index, float vec2s[]);
   public void get1Value(int index, SFVec2f vec);

   public void setValue(float vec2s[][]);
   public void setValue(float vec2s[]);
   public void setValue(int size, float vec2s[]);
   public void setValue(MFVec2f vecs);
   public void setValue(ConstMFVec2f vecs);

   public void set1Value(int index, float x, float y);
   public void set1Value(int index, ConstSFVec2f vec);
   public void set1Value(int index, SFVec2f vec);

   public void addValue(float x, float y);
   public void addValue(ConstSFVec2f vec);
   public void addValue(SFVec2f vec);

   public void insertValue(int index, float x, float y);
   public void insertValue(int index, ConstSFVec2f vec);
   public void insertValue(int index, SFVec2f vec);

   public String toString();   // This overrides a method in Object
}

public class MFVec3f extends MField
{
   public MFVec3f();
   public MFVec3f(float vec3s[][]);
   public MFVec3f(float vec3s[]);
   public MFVec3f(int size, float vec3s[]);

   public void getValue(float vec3s[][]);
   public void getValue(float vec3s[]);

   public void get1Value(int index, float vec3s[]);
   public void get1Value(int index, SFVec3f vec);

   public void setValue(float vec3s[][]);
   public void setValue(float vec3s[]);
   public void setValue(int size, float vec3s[]);
   public void setValue(MFVec3f vecs);
   public void setValue(ConstMFVec3f vecs);

   public void set1Value(int index, float x, float y, float z);
   public void set1Value(int index, ConstSFVec3f vec);
   public void set1Value(int index, SFVec3f vec);

   public void addValue(float x, float y, float z);
   public void addValue(ConstSFVec3f vec);
   public void addValue(SFVec3f vec);

   public void insertValue(int index, float x, float y, float z);
   public void insertValue(int index, ConstSFVec3f vec);
   public void insertValue(int index, SFVec3f vec);

   public String toString();   // This overrides a method in Object
}

public class ConstSFBool extends ConstField
{
   public ConstSFBool(boolean value);

   public boolean getValue();

   public String toString();   // This overrides a method in Object
}

public class ConstSFColor extends ConstField
{
   public ConstSFColor(float red, float green, float blue);

   public void getValue(float colors[]);
   public float getRed();
   public float getGreen();
   public float getBlue();

   public String toString();   // This overrides a method in Object
}

public class ConstSFFloat extends ConstField
{
   public ConstSFFloat(float value);

   public float getValue();

   public String toString();   // This overrides a method in Object
}

public class ConstSFImage extends ConstField
{
   public ConstSFImage(int width, int height, int components, byte pixels[]);

   public int getWidth();
   public int getHeight();
   public int getComponents();
   public void getPixels(byte pixels[]);

   public String toString();   // This overrides a method in Object
}

public class ConstSFInt32 extends ConstField
{
   public ConstSFInt32(int value);

   public int getValue();

   public String toString();   // This overrides a method in Object
}

public class ConstSFNode extends ConstField
{
   public ConstSFNode(BaseNode node);

   public BaseNode getValue();

   public String toString();   // This overrides a method in Object
}

public class ConstSFRotation extends ConstField
{
   public ConstSFRotation(float axisX, float axisY, float axisZ, float angle);

   public void getValue(float rotations[]);

   public String toString();   // This overrides a method in Object
}

public class ConstSFString extends ConstField
{
   public ConstSFString(String value);

   public String getValue();

   public String toString();   // This overrides a method in Object
}

public class ConstSFTime extends ConstField
{
   public ConstSFTime(double time);

   public double getValue();

   public String toString();   // This overrides a method in Object
}

public class ConstSFVec2f extends ConstField
{
   public ConstSFVec2f(float x, float y);

   public void getValue(float vec2s[]);
   public float getX();
   public float getY();

   public String toString();   // This overrides a method in Object
}

public class ConstSFVec3f extends ConstField
{
   public ConstSFVec3f(float x, float y, float z);

   public void getValue(float vec3s[]);
   public float getX();
   public float getY();
   public float getZ();

   public String toString();   // This overrides a method in Object
}

public class ConstMFColor extends ConstMField
{
   public ConstMFColor(float colors[][]);
   public ConstMFColor(float colors[]);
   public ConstMFColor(int size, float colors[]);

   public void getValue(float colors[][]);
   public void getValue(float colors[]);

   public void get1Value(int index, float colors[]);
   public void get1Value(int index, SFColor color);

   public String toString();   // This overrides a method in Object
}

public class ConstMFFloat extends ConstMField
{
   public ConstMFFloat(int size, float values[]);
   public ConstMFFloat(float values[]);

   public void getValue(float values[]);

   public float get1Value(int index);

   public String toString();   // This overrides a method in Object
}

public class ConstMFInt32 extends ConstMField
{
   public ConstMFInt32(int size, int values[]);
   public ConstMFInt32(int values[]);

   public void getValue(int values[]);

   public int get1Value(int index);

   public String toString();   // This overrides a method in Object
}

public class ConstMFNode extends ConstMField
{
   public ConstMFNode(int size, BaseNode node[]);
   public ConstMFNode(BaseNode node[]);

   public void getValue(BaseNode node[]);

   public BaseNode get1Value(int index);

   public String toString();   // This overrides a method in Object
}

public class ConstMFRotation extends ConstMField
{
   public ConstMFRotation(float rotations[][]);
   public ConstMFRotation(float rotations[]);
   public ConstMFRotation(int size, float rotations[]);

   public void getValue(float rotations[][]);
   public void getValue(float rotations[]);

   public void get1Value(int index, float rotations[]);
   public void get1Value(int index, SFRotation rotation);

   public String toString();   // This overrides a method in Object
}

public class ConstMFString extends ConstMField
{
   public ConstMFString(int size, String s[]);
   public ConstMFString(String s[]);

   public void getValue(String values[]);

   public String get1Value(int index);

   public String toString();   // This overrides a method in Object
}

public class ConstMFTime extends ConstMField
{
   public ConstMFTime(int size, double times[]);
   public ConstMFTime(double times[]);

   public void getValue(double times[]);

   public double get1Value(int index);

   public String toString();   // This overrides a method in Object
}

public class ConstMFVec2f extends ConstMField
{
   public ConstMFVec2f(float vec2s[][]);
   public ConstMFVec2f(float vec2s[]);
   public ConstMFVec2f(int size, float vec2s[]);

   public void getValue(float vec2s[][]);
   public void getValue(float vec2s[]);

   public void get1Value(int index, float vec2s[]);
   public void get1Value(int index, SFVec2f vec);

   public String toString();   // This overrides a method in Object
}

public class ConstMFVec3f extends ConstMField
{
   public ConstMFVec3f(float vec3s[][]);
   public ConstMFVec3f(float vec3s[]);
   public ConstMFVec3f(int size, float vec3s[]);

   public void getValue(float vec3s[][]);
   public void getValue(float vec3s[]);

   public void get1Value(int index, float vec3s[]);
   public void get1Value(int index, SFVec3f vec);

   public String toString();   // This overrides a method in Object
}

```

#### B.9.2.3 vrml.node Package

```
package vrml.node;

//
// This is the general Node class
//
public abstract class Node extends BaseNode
{
   // Get an EventIn by name. Return value is write-only.
   //   Throws an InvalidEventInException if eventInName isn't a valid
   //   eventIn name for a node of this type.
   public final Field getEventIn(String eventInName);

   // Get an EventOut by name. Return value is read-only.
   //   Throws an InvalidEventOutException if eventOutName isn't a valid
   //   eventOut name for a node of this type.
   public final ConstField getEventOut(String eventOutName);

   // Get an exposed field by name.
   //   Throws an InvalidExposedFieldException if exposedFieldName isn't a valid
   //   exposedField name for a node of this type.
   public final Field getExposedField(String exposedFieldName);

   public String toString();   // This overrides a method in Object
}

//
// This is the general Script class, to be subclassed by all scripts.
// Note that the provided methods allow the script author to explicitly
// throw tailored exceptions in case something goes wrong in the
// script.
//
public abstract class Script extends BaseNode
{
   // This method is called before any event is generated
   public void initialize();

   // Get a Field by name.
   //   Throws an InvalidFieldException if fieldName isn't a valid
   //   field name for a node of this type.
   protected final Field getField(String fieldName);

   // Get an EventOut by name.
   //   Throws an InvalidEventOutException if eventOutName isn't a valid
   //   eventOut name for a node of this type.
   protected final Field getEventOut(String eventOutName);

   // Get an EventIn by name.
   //   Throws an InvalidEventInException if eventInName isn't a valid
   //   eventIn name for a node of this type.
   protected final Field getEventIn(String eventInName);

   // processEvents() is called automatically when the script receives
   //   some set of events. It shall not be called directly except by its subclass.
   //   count indicates the number of events delivered.
   public void processEvents(int count, Event events[]);

   // processEvent() is called automatically when the script receives
   // an event.
   public void processEvent(Event event);

   // eventsProcessed() is called after every invocation of processEvents().
   public void eventsProcessed()

   // shutdown() is called when this Script node is deleted.
   public void shutdown();

   public String toString();   // This overrides a method in Object
}

```

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)B.10  Example of exception class

public class  **InvalidEventInException**
extends IllegalArgumentException

```
{
   /**
    * Constructs an InvalidEventInException with no detail message.
    */
   public InvalidEventInException(){
      super();
   }
   /**
    * Constructs an InvalidEventInException with the specified detail message.
    * A detail message is a String that describes this particular exception.
    * @param s the detail message
    */
   public InvalidEventInException(String s){
      super(s);
   }
}

public class InvalidEventOutException extends IllegalArgumentException
{
   public InvalidEventOutException(){
      super();
   }
   public InvalidEventOutException(String s){
      super(s);
   }
}

public class InvalidExposedFieldException extends IllegalArgumentException
{
   public InvalidExposedFieldException(){
      super();
   }
   public InvalidExposedFieldException(String s){
      super(s);
   }
}

public class InvalidFieldChangeException extends IllegalArgumentException
{
   public InvalidFieldChangeException(){
      super();
   }
   public InvalidFieldChangeException(String s){
      super(s);
   }
}

public class InvalidFieldException extends IllegalArgumentException
{
   public InvalidFieldException(){
      super();
   }
   public InvalidFieldException(String s){
      super(s);
   }
}

public class InvalidRouteException extends IllegalArgumentException
{
   public InvalidRouteException(){
      super();
   }
   public InvalidRouteException(String s){
      super(s);
   }
}

public class InvalidVRMLSyntaxException extends Exception
{
   public InvalidVRMLSyntaxException(){
      super();
   }
   public InvalidVRMLSyntaxException(String s){
      super(s);
   }

   public String getMessage();  // This overrides a method in Exception
}

```

![](../pix/vrmlbar.gif)

```
https://graphics.stanford.edu/courses/cs248-98-fall/Assignments/Assignment3/VRML2_Specification/spec/part1/java.html

```

