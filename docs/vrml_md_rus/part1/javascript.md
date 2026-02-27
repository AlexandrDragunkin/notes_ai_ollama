# ![](../pix/vrmllogo2.0.gif)

# The Virtual Reality Modeling Language

# Annex C. JavaScript Scripting Reference

### (normative)

### ISO/IEC DIS 14772-1

#### 4 April 1997

![](../pix/vrmlbar.gif)

## C.1 Introduction and table of contents

This annex describes the use of JavaScript with the [Script](nodesRef.html#Script) node. Subclause " [4.12 Scripting](concepts.html#4.12)" contains a general
overview of scripting in VRML while subclause " [6.40 Script](nodesRef.html#Script)" describes the Script
node.

[C.1 Introduction](#JavaScript Introduction)

[C.2 Language](#Language)

[C.3 Supported Protocol in the Script node's _url_\
field](#Protocol)

[C.3.1 \
Access](#Access)

[C.3.2 \
File Extension](#FileExtension)

[C.3.3 \
MIME Type](#MIMEtype)

[C.4 EventIn Handling](#EventIn)

[C.4.1 Receiving eventIns](#Receiving eventIns)

[C.4.2 \
Parameter passing and the EventIn Function](#ParameterPassing)

[C.4.3 \
eventsProcessed( ) Method](#EventsProcessed)

[C.4.4 \
initialize( ) Method](#Initialize)

[C.4.5 \
shutdown( ) Method](#Shutdown)

[C.5 Accessing Fields and Events](#AccessingFields)

[C.5.1 \
Accessing Fields and EventOuts of the Script](#AccessingScript)

[C.5.2 Accessing Fields and EventOuts of \
Other Nodes](#AccessingOtherNodes)

[C.5.3 \
Sending EventOuts](#SendingEventOuts)

[C.6 JavaScript Objects](#ExposedClasses)

[C.6.1 Notational conventions](#NotationalConventions)

[C.6.2 VRML Field to \
JavaScript variable conversion](#VRMLFieldToJavaScriptVariableConversion)

[C.6.3 \
Browser Object](#BrowserClass)

[C.6.4 \
SFColor object](#SFColor)

[C.6.5 \
SFImage object](#SFImage)

[C.6.6 \
SFNode object](#SFNode)

[C.6.7 \
SFRotation object](#SFRotation)

[C.6.8 \
SFVec2f object](#SFVec2f)

[C.6.9 \
SFVec3f object](#SFVec3f)

[C.6.10 \
MFColor object](#MFColor)

[C.6.11 \
MFFloat object](#MFFloat)

[C.6.12 \
MFInt32 object](#MFInt32)

[C.6.13 \
MFNode object](#MFNode)

[C.6.14 \
MFRotation object](#MFRotation)

[C.6.15 \
MFString object](#MFString)

[C.6.16 \
MFTime object](#MFTime)

[C.6.17 \
MFVec2f object](#MFVec2f)

[C.6.18 \
MFVec3f object](#MFVec3f)

[C.6.19 \
VrmlMatrix object](#VrmlMatrix)

[C.7 Examples](#Example)

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif) C.2 Language

[Netscape \
JavaScript](http://home.netscape.com/comprod/products/navigator/version_2.0/script/index.html) was created by Netscape Communications Corporation ( [http://home.netscape.com](http://home.netscape.com)).
JavaScript is a programmable API that allows cross-platform scripting
of events, objects, and actions. The JavaScript Specification, Version
1.1, can be found at [2.\[JAVS\]](references.html#[JAVS]). It
is expected that JavaScript, Version 1.2, will be the scripting
language of a Script node when JavaScript becomes a standard. Version
1.2 is required for VRML. The difference is that objects in numeric
expressions will have valueOf( ) called and if that fails, then
toString( ) will be called.

JavaScript is currently undergoing standardization through ECMA.

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif) C.3 Supported  protocol in the Script node's _url_ field

### C.3.1 Access

The _url_ field of the Script node may contain a URL that
references JavaScript code:

```
    Script { url "http://foo.com/myScript.js" }

```

The [javascript: protocol](concepts.html#4.5.5) allows the
script to be placed inline as follows:

```
    Script { url "javascript: function foo( ) { ... }" }

```

The _url_ field may contain multiple URL's and thus reference a
remote file or in-line code:

```
    Script {
      url [ "http://foo.com/myScript.js",
      "javascript: function foo( ) { ... }" ]
    }

```

### C.3.2 File extension

The file extension for JavaScript source code is `.js`.

### C.3.3 MIME type

The MIME type for JavaScript source code is defined as follows:

```
     application/x-javascript

```

## ![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif) C.4 EventIn  Handling

### C.4.1 Receiving eventIns

Events sent to the Script node are passed to the corresponding
JavaScript function in the script. The script is specified in the _url_
field of the Script node. The function's name is the same as the
eventIn and is passed two arguments, the event value and its timestamp
(see " [C.4.2 Parameter passing and the \
EventIn function"](#ParameterPassing)). If there is no corresponding JavaScript
function in the script, the browser's behaviour is undefined.

For example, the following Script node has one eventIn field whose name
is _start_:

```
    Script {
      eventIn SFBool start
      url "javascript: function start(value, timestamp) { ... }"
    }

```

In the above example, when the _start_ eventIn is sent, the start(
) function is executed.

### C.4.2 Parameter passing and the eventIn  function

When a Script node receives an eventIn, a corresponding method in the
file specified in the _url_ field of the Script node is called.
This method has two arguments. The value of the eventIn is passed as
the first argument and the timestamp of the eventIn is passed as the
second argument. The type of the value is the same as the type of the
eventIn and the type of the timestamp is SFTime. " [C.6.1 VRML Field to \
JavaScript variable conversion"](#VRMLFieldToJavaScriptVariableConversion) provides a description
of how VRML types appear in JavaScript.

### C.4.3 eventsProcessed( ) method

Authors may define a function named eventsProcessed( ) which is to
be called after some set of events has been received. Some
implementations call this function after the return from each EventIn
function, while others call it only after processing a number of
EventIn functions. In the latter case, an author can improve
performance by placing lengthy processing algorithms which do not need
to execute for every event received into the eventsProcessed( )
function.

The eventsProcessed( ) function takes no parameters. Events
generated from it are given the timestamp of the last event processed.

### C.4.4 initialize( )  method

Authors may define a function named initialize( ) which is invoked **before**
the browser presents the world to the user and **before**
any events are processed by any nodes in the same VRML file as the
Script node containing this script (see " [4.12.3 Initialize() and shutdown()](concepts.html#4.12.3)").

The initialize( ) function has no parameters. Events generated
from initialize( ) are given the timestamp of when the Script node
was loaded.

### C.4.5 shutdown( ) method

Authors may define a function named shutdown( ) which is invoked
when the corresponding Script node is deleted or when the world
containing the Script node is unloaded or replaced by another world (see " [4.12.3 Initialize() and shutdown()](concepts.html#4.12.3)").

The shutdown( ) function has no parameters. Events generated from shutdown( )
are given the timestamp of when the Script node was deleted.

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif) C.5 Accessing fields

### C.5.1 Accessing Fields and EventOuts of the  Script

The fields and eventOuts of a Script node are accessible from its
JavaScript functions. As in all other nodes, the fields are accessible
only within the Script. The eventIns are not accessible. The Script
node's eventIns can be routed to and its eventOuts can be routed from.
Another Script node with a pointer to this node can access its eventIns
and eventOuts as for any other node.

Fields defined in the Script node are available to the script by using
its name. Its value can be read or written. This value is persistent
across function calls. EventOuts defined in the script node can also be
read. The value is the last value assigned.

### C.5.2 Accessing fields and eventOuts of  other nodes

The script can access any exposedField, eventIn or eventOut of any node
to which it has a pointer:

```
    DEF SomeNode Transform { }
    Script {
      field SFNode node USE SomeNode
      eventIn SFVec3f pos
      directOutput TRUE
      url "javascript:
        function pos(value) {
          node.set_translation = value;
        }"
    }

```

This example sends a set\_translation eventIn to the Transform node. An
eventIn on a passed node can appear only on the left side of the
assignment. An eventOut in the passed node can appear only on the right
side, which reads the last value sent out. Fields in the passed node
cannot be accessed. However, exposedFields can either send an event to
the " _set\__..." eventIn or read the current value of the
"... _\_changed_" eventOut. This follows the routing model
of the rest of VRML.

Events generated by setting an eventIn on a node are sent at the
completion of the currently executing function. The eventIn shall be
assigned a value of the same datatype; no partial assignments are
allowed. For example, it is not possible to assign the red value of an
SFColor eventIn. Since eventIns are strictly write-only, the remainder
of the partial assignment would have invalid field values. Assigning to
the eventIn field multiple times during one execution of the function
still only sends one event and that event is the last value assigned.

### C.5.3 Sending eventOuts

Assigning to an eventOut sends that event at the completion of the
currently executing function. Assigning to the eventOut multiple times
during one execution of the function still only sends one event and
that event is the value of the eventOut at the completion of script
execution.

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif) C.6 JavaScript objects

### C.6.1 Notational conventions

Since JavaScript is an untyped language it has no language constructs
to describe the types of parameters passed to, or values returned from
methods. Therefore this document uses a notational convention to
describe these types. Parameters passed are preceded by their type, and
the type of any return value precedes the method name. Normally these
types correspond to VRML field types, so those names are used. In the
case of no return value, the identifier _void_ is used. In the
case of a JavaScript numeric value or numeric array return, the
identifier _numeric_ or _numeric\[ \]_ is used. In the case of
a string return, the identifier _String_ is used.

### C.6.2 VRML Field to  JavaScript variable conversion

JavaScript native datatypes consist of boolean, numeric and string. The
language is not typed, so datatypes are implicit upon assignment. The
VRML SFBool is mapped to the JavaScript boolean. In addition to the
JavaScript _true_ and _false_ constants, the VRML TRUE and
FALSE values may be used. The VRML SFInt32, SFFloat and SFTime fields
are mapped to the numeric datatype and will be maintained in double
precision accuracy. These types are passed by value in function calls.
All other VRML fields are mapped to JavaScript objects. JavaScript
objects are passed by reference.

The JavaScript boolean, numeric and string are automatically converted
to other datatypes when needed. See [2.\[JAVS\]](references.html#[JAVS])
for more details.

In JavaScript, assigning a new value to a variable gives the variable
the datatype of the new value, in addition to the value. Scalar values
(boolean and numeric) are assigned by copying the value. Other
objects are assigned by reference.

When assignments are made to eventOuts and fields, the values are
converted to the VRML field type. Scalar values (boolean and numeric)
are assigned by copying the value. Other objects are assigned by
reference. There is an [example on assigning](#assigning with)
to illustrate the differences.

The SF objects will be assigned as references, except for assigning to
or from eventOut, fields, and MF objects. The exceptions for eventOut,
field and MF objects are at the interface between VRML field values and
JavaScript variables. The VRML fields are maintained in the correct
datatype and are copied during assignment.

For eventOut objects, assignment copies the value to the eventOut,
which will be sent upon completion of the current function. Assigning
an eventOut to an internal variable creates a new object of the same
type as the eventOut with the current value of the eventOut. Field
objects behave identically to eventOut objects, except that no event is
sent upon completion of the function.

Assigning an element of an MF object to an SF object creates a new
object of the corresponding SF object type with the current value of
the specified MF element. Assigning an SF object to an element of an MF
object (which shall be of the corresponding type) copies the value of
the SF object into the dereferenced element of the MF object.

### C.6.3 Browser object

This subclause lists the class static methods available in the _Browser_ object
which allow scripts to get and set browser information. Descriptions of
the methods are provided in " [4.12.10 Browser script interface"](concepts.html#4.12.10).
The syntax for a call is:

```
    mymfnode = Browser.createVrmlFromString('Sphere {}');

```

Table C.1 describes the Browser object's methods, parameters, and
return values.

#### Table C.1: Browser object functions

**Return value****Method**
 String**getName**( )String**getVersion**( )numeric**getCurrentSpeed**( )numeric**getCurrentFrameRate**( )String**getWorldURL**( )void**replaceWorld**( MFNode nodes )MFNode**createVrmlFromString**( String vrmlSyntax )void**createVrmlFromURL**( MFString url, Node node,
 String event )void**addRoute**( SFNode fromNode, String
 fromEventOut,

                       SFNode
 toNode, String toEventIn)void**deleteRoute**( SFNode fromNode, String
 fromEventOut,

                          SFNode
 toNode, String toEventIn )void**loadURL**( MFString url, MFString parameter )void**setDescription**( String description )

### C.6.4 SFColor object

#### C.6.4.1 Description

The SFColor object corresponds to a VRML SFColor field. All properties
are accessed using the syntax _sfColorObjectName.<property>_,
where _sfColorObjectName_ is an instance of an SFColor object. The
properties may also be accessed by the indices \[0\] for red, \[1\] for
green and \[2\] blue. All methods are invoked using the syntax _sfColorObjectName.method(<argument-list>)_,
where _sfColorObjectName_ is an instance of an SFColor object.

#### C.6.4.2 Instance creation method

_sfColorObjectName =_ new SFColor(float _r,_ float _g,_ float _b)_

where

_r, g,_ and _b_ are the red, green, and blue values of the
colour. Missing values will be filled by 0.0.

#### C.6.4.3 Properties

The properties of the SFColor object are described in Table C.2.

#### Table C.2: SFColor properties

**Property****Description**numeric _r_red component of the colour numeric _g_green component of the colour numeric _b_blue component of the colour

#### C.6.4.4 Methods

The methods of the SFColor object are described in Table C.3.

#### **Table C.3: SFColor methods**

**Method**

**Description**

void setHSV(float _h_, float _s,_ float _v_) Sets the value of the colour by specifying the values of _hue_, _saturation_,
 and _value_. numeric\[3\] getHSV( ) Returns the value of the colour in a 3 element numeric
 array, with _hue_ at index 0, _saturation_ at
 index 1, and _value_ at index 2. String toString( ) Returns a String containing the VRML 97 utf8 encoded value
 of _r_, _g_ and _b_.

### C.6.5 SFImage object

#### C.6.5.1 Description

The SFImage object corresponds to a VRML SFImage field.

#### C.6.5.2 Instance creation method

_sfImageObjectName =_ new SFImage(numeric _x,_ numeric _y,_ numeric _comp,_ MFInt32 _array)_

where

_x_ is the x-dimension of the image. _y_ is the y-dimension
of the image. _comp_ is the number of components of the image (1
for greyscale, 2 for greyscale+alpha, 3 for rgb, 4 for rgb+alpha). _Array_
contains the _x_ x _y_ values for the pixels of the image.
Format of each pixel is the same as the [**PixelTexture**](nodesRef.html#PixelTexture) file format.

#### C.6.5.3 Properties

The properties of the SFImage object are listed in Table C.4.

#### Table C.4: SFImage properties

**Property****Description**numeric _x_x dimension of the imagenumeric _y_y dimension of the imagenumeric _comp_
 number of components of the image:

 1: greyscale

 2: greyscale + alpha

 3: rgb

 4: rgb + alpha

 MFInt32 _array_image data

#### C.6.5.4 Methods

The method of the SFImage object is described in Table C.5.

#### Table C.5: SFImage method

**Method****Description**String toString( )Returns a String containing the VRML 97 UTF-8 encoded value
 of x, y, comp and array.

### C.6.6 SFNode object

#### C.6.6.1 Description

The SFNode object corresponds to a VRML SFNode field.

#### C.6.6.2 Instance creation method

_sfNodeObjectName =_ new SFNode(String _vrmlstring)_

where

_vrmlstring_ is an ISO 646 string containing the definition
of a VRML node

#### C.6.6.3 Properties

Each node may assign values to its eventIns and obtain the last output
values of its eventOuts using the _sfNodeObjectName.eventName_
syntax.

#### C.6.6.4 Methods

The method of the SFNode object is described in Table C.6.

#### Table C.6: SFNode method

**Method****Description**String toString( )Returns the VRML utf8 string that, if parsed as the value
 of an SFNode field, would produce this node. If the browser is unable
 to reproduce this node, the name of the node followed by the open brace
 and close brace shall be returned. Additional information may be
 included as one or more VRML comment strings.

### C.6.7 SFRotation object

#### C.6.7.1 Description

The SFRotation object corresponds to a VRML SFRotation field. It has
four numeric properties: x, y, z (the axis of rotation) and angle.
These may also be addressed by indices \[0\] through \[3\].

#### C.6.7.2 Instance creation methods

_sfRotationObjectName =_ new SFRotation(numeric _x,_ numeric _y,_ numeric _z,_ numeric _angle)_

where

_x_, _y_, and _z_ are the axis of the rotation. _angle_
is the angle of the rotation (in radians). Missing values default to
0.0, except _y_, which defaults to 1.0.

_sfRotationObjectName_ = new SFRotation(SFVec3f _axis,_ numeric _angle_)

where

_axis_ is the axis of rotation. _angle_ is the angle of the
rotation (in radians)

_sfRotationObjectName_ = new SFRotation(SFVec3f _fromVector,_ SFVec3f _toVector_)

where

_fromVector_ and _toVector_ are normalized and the rotation
value that would rotate from the _fromVector_ to the _toVector_
is stored in the object.

#### C.6.7.3 Properties

The properties of the SFRotation object are described in Table C.7.

#### Table C.7: SFRotation properties

**Property****Description**numeric _x_first value of the axis vectornumeric _y_second value of the axis vectornumeric _z_third value of the axis vectornumeric _angle_the angle of the rotation (in radians)

#### C.6.7.4 Methods

The methods of the SFRotation object are described in Table C.8.

#### Table C.8: SFRotation methods

**Method****Description**SFVec3f getAxis( )Returns the axis of rotation.SFRotation inverse( )Returns the inverse of this object's rotation.SFRotation multiply(SFRotation _rot_)Returns the object multiplied by the passed value.SFVec3f multVec(SFVec3f _vec_)Returns the value of _vec_ multiplied by the matrix
 corresponding to this object's rotation.void setAxis(SFVec3f _vec_)Sets the axis of rotation to the value passed in _vec_.SFRotation slerp(SFRotation _dest,_ numeric _t_) Returns the value of the spherical linear interpolation
 between this object's rotation and _dest_ at value 0
 <= _t_ <= 1\. For _t_ = 0, the value is this
 object's rotation. For _t_ = 1, the value is _dest_.String toString( )Returns a String containing the VRML 97 utf8 encoded value
 of x, y, z, and angle.

### C.6.8 SFVec2f object

#### C.6.8.1 Description

The SFVec2f object corresponds to a VRML SFVec2f field. Each component
of the vector can be accessed using the _x_ and _y_
properties or using C-style array dereferencing ( _i. e., sfVec2fObjectName\[0\]_ or _sfVec2fObjectName\[1\])._

#### C.6.8.2 Instance creation method

_sfVec2fObjectName =_ new SFVec2f(numeric _x,_ numeric _y)_

Missing values default to 0.0.

#### C.6.8.3 Properties

The properties of the SFVec2f object are described in Table C.9.

#### Table C.9: SFVec2f properties

**Property****Description**numeric _x_First value of the vector.numeric _y_Second value of the vector.

#### C.6.8.4 Methods

The methods of the SFVec2f object are described in Table C.10.

#### Table C.10: SFVec2f methods

**Method****Description**SFVec2f add(SFVec2f _vec_)Returns the value of the passed value added,
 component-wise, to the object.SFVec2f divide(numeric _n_)Returns the value of the object divided by the passed value.numeric dot(SFVec2f _vec_)Returns the dot product of this vector and the passed value.numeric length( )Returns the geometric length of this vector.SFVec2f multiply(numeric _n_)Returns the value of the object multiplied by the passed
 value.SFVec2f normalize( )Returns the object converted to unit length .SFVec2f subtract(SFVec2f _vec_)Returns the value of the passed value subtracted,
 component-wise, from the object.String toString( )Returns a String containing the VRML 97 utf8 encoded value
 of x and y.

### C.6.9 SFVec3f object

#### C.6.9.1 Description

The SFVec3f object corresponds to a VRML SFVec3f field. Each component
of the vector can be accessed using the x, y, and z properties or using
C-style array dereferencing ( _i. e., sfVec3fObjectName\[0\],_
_sfVec3fObjectName\[1\]_ or _sfVec3fObjectName\[2\])._

#### C.6.9.2 Instance creation method

_sfVec3fObjectName =_ new SFVec3f(numeric _x,_ numeric _y,_ numeric _z)_

Missing values default to 0.0.

#### C.6.9.3 Properties

The properties of the SFVec3f object are described in Table C.11.

#### Table C.11: SFVec2f properties

**Property****Description**numeric _x_First value of the vector. numeric _y_Second value of the vector. numeric _z_Third value of the vector.

#### C.6.9.4 Methods

The methods of the SFVec3f object are described in Table C.12.

#### Table C.12: SFVec3f methods

**Method****Description**SFVec3f add(SFVec3f _vec_)Returns the value of the passed value added,
 component-wise, to the object.SFVec3f cross(SFVec3f _vec_)Returns the cross product of the object and the passed
 value.SFVec3f divide(numeric _n_)Returns the value of the object divided by the passed value.numeric dot(SFVec3f _vec_)Returns the dot product of this vector and the passed value.numeric length( )Returns the geometric length of this vector.SFVec3f multiply( _numeric n_)Returns the value of the object multiplied by the passed
 value.SFVec3f negate( )Returns the value of the component-wise negation of the
 object.SFVec3f normalize( )Returns the object converted to unit length .SFVec3f subtract(SFVec3f _vec_)Returns the value of the passed value subtracted,
 component-wise, from the object.String toString( )Returns a String containing the VRML 97 utf8 encoded value
 of x, y, and z.

### C.6.10 MFColor object

#### C.6.10.1 Description

The MFColor object corresponds to a VRML MFColor field. It is used to
store a one-dimensional array of SFColor objects. Individual elements
of the array can be referenced using the standard C-style dereferencing
operator ( _e. g._, _mfColorObjectName_\[ _index_\],
where _index_ is an integer-valued expression with 0 <= _index_ <
length and length is the number of elements in the array). Assigning to
an element with _index_ \> length results in the array being
dynamically expanded to contain length elements. All elements not
explicitly initialized are set to SFColor (0, 0, 0).

#### C.6.10.2 Instance creation method

_mfColorObjectName =_ new MFColor(SFColor _c1,_ SFColor _c2, ...)_

The creation method shall initialize the array using 0 or more
SFColor-valued expressions passed as parameters.

#### C.6.10.3 Property

The property of the MFColor object is described in Table C.13.

#### Table C.13: MFColor properties

**Property****Description**numeric _length_property for getting/setting the number of elements in the
 array.

#### C.6.10.4 Method

The method of the MFColor object is described in Table C.14.

#### Table C.14: MFColor methods

**Method****Description**String toString( )Returns a String containing the VRML 97 utf 8 encoded value
 of the MFColor array.

### C.6.11 MFFloat object

#### C.6.11.1 Description

The MFFloat object corresponds to a VRML MFFloat field. It is used to
store a one-dimensional array of SFFloat values. Individual elements of
the array can be referenced using the standard C-style dereferencing
operator ( _e. g._, _mfFloatObjectName_\[ _index_\],
where _index_ is an integer-valued expression with 0 <= _index_ <
length and length is the number of elements in the array). Assigning to
an element with _index_ \> length results in the array being
dynamically expanded to contain length elements. All elements not
explicitly initialized are set to 0.0.

#### C.6.11.2 Instance creation method

_mfFloatObjectName =_ new MFFloat(numeric _n1,_ numeric _n2, ...)_

where

The creation method shall initialize the array using 0 or more
numeric-valued expressions passed as parameters.

#### C.6.11.3 Property

The property of the MFFloat object is described in Table C.15.

#### Table C.15: MFFloat properties

**Property****Description**numeric _length_property for getting/setting the number of elements in the
 array.

#### C.6.11.4 Method

The method of the MFFloat object is described in Table C.16.

#### Table C.16: MFFloat method

**Method****Description**String toString( ) Returns a String containing the VRML 97 utf 8 encoded value
 of the MFFloat array.

### C.6.12 MFInt32 object

#### C.6.12.1 Description

The MFInt32 object corresponds to a VRML MFInt32 field. It is used to
store a one-dimensional array of SFInt32 values. Individual elements of
the array can be referenced using the standard C-style dereferencing
operator ( _e. g._, _mfInt32ObjectName_\[ _index\]_,
where _index_ is an integer-valued expression with 0 <= _index_ <
length and length is the number of elements in the array). Assigning to
an element with _index_ \> length results in the array being
dynamically expanded to contain length elements. All elements not
explicitly initialized are set to 0.

#### C.6.12.2 Instance creation method

_mfInt32ObjectName =_ new MFInt32(numeric _n1,_ numeric _n2, ...)_

where

The creation method shall initialize the array using 0 or more
integer-valued expressions passed as parameters.

#### C.6.12.3 Property

The property of the MFInt32 object is described in Table C.17.

#### Table C.17: MFInt32 property

**Property****Description**numeric _length_property for getting/setting the number of elements in the
 array.

#### C.6.12.4 Method

The method of the MFInt32 object is described in Table C.18.

#### Table C.18: MFInt32 method

**Method****Description**String toString( ) Returns a String containing the VRML 97 utf 8 encoded value
 of the MFInt32 array.

### C.6.13 MFNode object

#### C.6.13.1 Description

The MFNode object corresponds to a VRML MFNode field. It is used to
store a one-dimensional array of SFNode objects. Individual elements of
the array can be referenced using the standard C-style dereferencing
operator ( _e. g._, _mfNodeObjectName_\[ _index_\],
where _index_ is an integer-valued expression with 0 <= _index_ <
length and length is the number of elements in the array). Assigning to
an element with _index_ \> length results in the array being
dynamically expanded to contain length elements. All elements not
explicitly initialized are set to NULL.

#### C.6.13.2 Instance creation method

_mfNodeObjectName =_ new MFNode(SFNode _n1,_ SFNode _n2,_
_...)_

where

The creation method shall initialize the array using 0 or more
SFNode-valued expressions passed as parameters.

#### C.6.13.3 Property

The property of the MFNode object is described in Table C.19.

#### Table C.19: MFNode property

**Property****Description**numeric _length_property for getting/setting the number of elements in the
 array.

#### C.6.13.4 Method

The method of the MFNode object is described in Table C.20.

#### Table C.20: MFNode method

**Method****Description**String toString( )Returns the VRML utf8 string that, if parsed as the value
 of a MFNode field, would produce this array of nodes. If the browser is
 unable to reproduce this node, the name of the node followed by the
 open brace and close brace shall be returned. Additional information
 may be included as one or more VRML comment strings

### C.6.14 MFRotation object

#### C.6.14.1 Description

The MFRotation object corresponds to a VRML MFRotation field. It is
used to store a one-dimensional array of SFRotation objects. Individual
elements of the array can be referenced using the standard C-style
dereferencing operator ( _e. g._, _mfRotationObjectName_\[ _index_\],
where _index_ is an integer-valued expression with 0 <= _index_ <
length and length is the number of elements in the array). Assigning to
an element with _index_ \> length results in the array being
dynamically expanded to contain length elements. All elements not
explicitly initialized are set to SFRotation (0, 0, 1, 0).

#### C.6.14.2 Instance creation method

_mfRotationObjectName =_ new MFRotation(SFRotation _r1,_ SFRotation _r2, ...)_

where

The creation method shall initialize the array using 0 or more
SFRotation-valued expressions passed as parameters.

#### C.6.14.3 Property

The property of the MFRotation object is described in Table C.21.

#### Table C.21: MFRotation property

**Property****Description**numeric _length_property for getting/setting the number of elements in the
 array.

#### C.6.14.4 Method

The method of the MFRotation object is described in Table C.22.

#### Table C.22: MFRotation method

**Method****Description**String toString( ) Returns a String containing the VRML 97 utf 8 encoded value
 of the MFRotation array.

### C.6.15 MFString Object

#### C.6.15.1 Description

The MFString object corresponds to a VRML 2.0 MFString field. It is
used to store a one-dimensional array of String objects. Individual
elements of the array can be referenced using the standard C-style
dereferencing operator ( _e. g._, _mfStringObjectName_\[ _index_\],
where _index_ is an integer-valued expression with 0 <= _index_ <
length and length is the number of elements in the array). Assigning to
an element with _index_ \> length results in the array being
dynamically expanded to contain length elements. All elements not
explicitly initialized are set to the empty string.

#### C.6.15.2 Instance creation method

_mfStringObjectName =_ new MFString(String _s1,_ String _s2, ...)_

where

The creation method shall initialize the array using 0 or more
String-valued expressions passed as parameters.

#### C.6.15.3 Property

The property of the MFString object is described in Table C.23.

#### Table C.23: MFString property

**Property****Description**numeric _length_property for getting/setting the number of elements in the
 array.

#### C.6.15.4 Method

The method of the MFString object is described in Table C.24.

#### Table C.24: MFString method

**Method****Description**String toString( )Returns a String containing the VRML 97 utf 8 encoded value
 of the MFString array.

### C.6.16 MFTime object

#### C.6.16.1 Description

The MFTime object corresponds to a VRML MFTime field. It is used to
store a one-dimensional array of SFTime values. Individual elements of
the array can be referenced using the standard C-style dereferencing
operator ( _e. g._, _mfTimeObjectName_\[ _index_\],
where _index_ is an integer-valued expression with 0 <= _index_ <
length and length is the number of elements in the array). Assigning to
an element with _index_ \> length results in the array being
dynamically expanded to contain length elements. All elements not
explicitly initialized are set to 0.0.

#### C.6.16.2 Instance creation method

_mfTimeObjectName =_ new MFTime(numeric _n1,_ numeric _n2,_
_...)_

The creation method shall initialize the array using 0 or more
numeric-valued expressions passed as parameters.

#### C.6.16.3 Property

The property of the MFTime object is described in Table C.25.

#### Table C.25: MFTime property

**Property****Description**numeric _length_property for getting/setting the number of elements in the
 array.

#### C.6.16.4 Method

The method of the MFTime object is described in Table C.26.

#### Table C.26: MFTime method

**Method****Description**String toString( )Returns a String containing the VRML 97 utf 8 encoded value
 of the MFTime array.

### C.6.17 MFVec2f object

#### C.6.17.1 Description

The MFVec2f object corresponds to a VRML MFVec2f field. It is used to
store a one-dimensional array of SFVec2f objects. Individual elements
of the array can be referenced using the standard C-style dereferencing
operator ( _e. g._, _mfVec2fObjectName_\[ _index_\],
where _index_ is an integer-valued expression with 0 <= _index_ <
length and length is the number of elements in the array). Assigning to
an element with _index_ \> length results in the array being
dynamically expanded to contain length elements. All elements not
explicitly initialized are set to SFVec2f (0, 0).

#### C.6.17.2 Instance creation method

_mfVec2fObjectName =_ new MFVec2f(SFVec2f _v1,_ SFVec2f _v2, ...)_

The creation method shall initialize the array using 0 or more
SFVec2f-valued expressions passed as parameters.

#### C.6.17.3 Property

The property of the MFVec2f object is described in Table C.27.

#### Table C.27: MFVec2f property

**Property****Description**numeric _length_property for getting/setting the number of elements in the
 array.

#### C.6.17.4 Method

The method of the MFVec2f object is described in Table C.28.

#### Table C.28: MFVec2f method

**Method****Description**String toString( )Returns a String containing the VRML 97 utf 8 encoded value
 of the MFVec2f array.

### C.6.18 MFVec3f object

#### C.6.18.1 Description

The MFVec3f object corresponds to a VRML MFVec3f field. It is used to
store a one-dimensional array of SFVec3f objects. Individual elements
of the array can be referenced using the standard C-style dereferencing
operator ( _e. g._, _mfVec3fObjectName_\[ _index_\],
where _index_ is an integer-valued expression with 0 <= _index_ <
length and length is the number of elements in the array). Assigning to
an element with _index_ \> length results in the array being
dynamically expanded to contain length elements. All elements not
explicitly initialized are set to SFVec3f (0, 0, 0).

#### C.6.18.2 Instance creation method

_mfVec3fObjectName =_ new MFVec3f(SFVec3f _v1,_ SFVec3f _v2,...)_

where

The creation method shall initialize the array using 0 or more
SFVec3f-valued expressions passed as parameters.

#### C.6.18.3 Property

The property of the MFVec3f object is described in Table C.29.

#### Table C.29: MFVec3f property

**Property****Description**numeric _length_property for getting/setting the number of elements in the
 array.

#### C.6.18.4 Method

The method of the MFVec3f object is described in Table C.30.

#### Table C.30: MFVec3f method

**Method****Description**String toString( )Returns a String containing the VRML 97 utf 8 encoded value
 of the MFVec3f array.

### C.6.19 VrmlMatrix Object

#### C.6.19.1 Description

The VrmlMatrix object provides many useful methods for performing
manipulations on 4x4 matrices. Each of element of the matrix can be
accessed using C-style array dereferencing ( _i. e.,_ vrmlMatrixObjectName\[0\]\[1\]
is the element in row 0, column 1). The results of dereferencing a
VrmlMatrix object using a single index ( _i. e.,_ vrmlMatrixObjectName\[0\])
are undefined. The translation elements are in the fourth row. For
example, vrmlMatrixObjectName\[3\]\[0\] is the X offset.

#### C.6.19.2 Instance creation methods

_VrmlMatrixObjectName =_ new VrmlMatrix(

                                                                       numeric _f11,_ numeric _f12,_ numeric _f13,_ numeric _f14,_

                                                                       numeric _f21,_ numeric _f22,_ numeric _f23,_ numeric _f24,_

                                                                       numeric _f31,_ numeric _f32,_ numeric _f33,_ numeric _f34,_

                                                                       numeric _f41,_ numeric _f42,_ numeric _f43,_ numeric _f44)_

A new matrix initialized with the values in _f11_ through _f44_
is created and returned. The translation values will be _f41, f42_,
and _f43_.

_VrmlMatrixObjectName =_ new VrmlMatrix( )

A new matrix initialized with the identity matrix is created and
returned.

#### C.6.19.3 Properties

The VRMLMatrix object has no properties.

#### C.6.19.4 Methods

The methods of the VRMLMatrix object are listed in Table C.31.

#### Table C.31: VRMLMatrix methods

**Method****Description**void setTransform(SFVec3f _translation_,

                         SFRotation _rotation_,

                         SFVec3f _scale_,

                         SFRotation _scaleOrientation_,

                         SFVec3f _center_) Sets the VrmlMatrix to the passed values. Any of the
 rightmost parameters may be omitted. The method has 0 to 5 parameters.
 For example, specifying 0 parameters results in an identity matrix
 while specifying 1 parameter results in a translation and specifying 2
 parameters results in a translation and a rotation. Any unspecified
 parameter is set to its default as specified for the [**Transform**](nodesRef.html#Transform) node. void getTransform(SFVec3f _translation_,

                          SFRotation _rotation_,

SFVec3f _scale_) Decomposes the VrmlMatrix and returns the components in the
 passed _translation_, _rotation_, and _scale_ objects _._ The
 types of these passed objects is the same as the first three arguments
 to **setTransform**. If any passed object is not sent,
 or if the null object is sent for any value, that value is not
 returned. Any projection or shear information in the matrix is ignored. VrmlMatrix inverse( ) Returns a VrmlMatrix whose value is the inverse of this
 object. VrmlMatrix transpose( ) Returns a VrmlMatrix whose value is the transpose of this
 object. VrmlMatrix multLeft(VrmlMatrix _matrix_) Returns a VrmlMatrix whose value is the object multiplied
 by the passed _matrix_ on the left. VrmlMatrix multRight(VrmlMatrix _matrix_) Returns a VrmlMatrix whose value is the object multiplied
 by the passed _matrix_ on the right. SfVec3f multVecMatrix(SFVec3f _vec_) Returns an SFVec3f whose value is the object multiplied by
 the passed row vector. SFVec3f multMatrixVec(SFVec3f _vec_) Returns an SFVec3f whose value is the object multiplied by
 the passed column vector. String toString( ) Returns a String containing the values of the VrmlMatrix.

## ![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif) C.7 Examples

The following is an example of a [Script](nodesRef.html#Script)
node which determines whether a given colour contains a lot of red.
The Script node exposes a Color field, an eventIn, and an eventOut:

```
DEF Example_1 Script {
        field    SFColor currentColor 0 0 0
    eventIn  SFColor colorIn
    eventOut SFBool  isRed

    url "javascript:
        function colorIn(newColor, ts) {
            // This method is called when a colorIn event is received
            currentColor = newColor;
        }

        function eventsProcessed( ) {
            if (currentColor[0] >= 0.5)
                // if red is at or above 50%
                isRed = true;
        }"
}

```

Details on when the methods defined in Example\_2 Script are called are
provided in " [4.12.2 Script \
Execution"](concepts.html#4.12.2).

The following example illustrate use of the createVrmlFromURL( )
method:

```
 DEF Example_2 Script {
    field   SFNode myself USE Example_2
    field   SFNode root USE ROOT_TRANSFORM
    field   MFString url "foo.wrl"
    eventIn MFNode   nodesLoaded
    eventIn SFBool   trigger_event

    url "javascript:
        function trigger_event(value, ts){
            // do something and then fetch values
            Browser.createVRMLFromURL(url, myself, 'nodesLoaded');
        }

        function nodesLoaded(value, timestamp){
            if (value.length > 5) {
                 // do something more than 5 nodes in this MFNode...
            }
            root.addChildren = value;
        }"
}

```

The following example illustrates use of the addRoute( ) method:

```

DEF Sensor TouchSensor {}

DEF Baa Script {
    field   SFNode myself USE Baa
    field   SFNode fromNode USE Sensor
    eventIn SFBool clicked
    eventIn SFBool trigger_event

    url "javascript:
        function trigger_event(eventIn_value){
            // do something and then add routing
            Browser.addRoute(fromNode, 'isActive', myself, 'clicked');
        }

        function clicked(value){
            // do something
        }"
}

```

The following example illustrates assigning with references and
assigning by copying:

```
Script {
    eventIn  SFBool  eI
    eventOut SFVec3f eO
    field    MFVec3f f []

    url "javascript:
        function eI( ) {
            eO = new SFVec3f(0,1,2);  // 'eO' contains the value
                                      // (0,1,2) which will be sent
                                      // out when the function
                                      // is complete.
            a = eO;                   // 'a' contains a SFVec3f
                                      // object with the value (0,1,2)
            b = a;                    // 'b' references the same
                                      // object as 'a'.
            a.x = 3;                  // 'a' and 'b' both contain
                                      // (3,1,2). 'eO' is unchanged.
            f[1] = a;                 // 'f[1]' contains the value
                                      // (3,1,2).
            c = f[1];                 // 'b' contains a SFVec3f
                                      // object with the value (3,1,2)
            f[1].y = 4;               // 'f[1]' contains the value
                                      // (3,4,2). 'c' is unchanged.
        }"
}

```

The following example illustrates uses of the fields and methods of
SFVec3f and MFVec3f:

```
DEF SCR-VEC3F Script {
    eventIn SFTime touched1
    eventIn SFTime touched2
    eventIn SFTime touched3
    eventIn SFTime touched4
    eventOut SFVec3f new_translation
    field SFInt32 count 1
    field MFVec3f verts  []

    url "javascript:
        function initialize( ) {
            verts[0] = new SFVec3f(0, 0, 0);
            verts[1] = new SFVec3f(1, 1.732, 0);
            verts[2] = new SFVec3f(2, 0, 0);
            verts[3] = new SFVec3f(1, 0.577, 1.732);
        }

        function touched1 (value) {
            new_translation = verts[count]; // move sphere around tetra
            count++;
            if (count >= verts.length) count = 1;
        }

        function touched2 (value) {
            var tVec;
            tVec = new_translation.divide(2); // Zeno's paradox to origin
            new_translation = new_translation.subtract(tVec);
        }

        function touched4 (value) {
            new_translation = new_translation.negate( );
        }

        function touched3 (value) {
            var a;
            a = verts[1].length( );
            a = verts[3].dot(verts[2].cross(verts[1]));
            a = verts[1].x;
            new_translation = verts[2].normalize( );
            new_translation = new_translation.add(new_translation);
        }"
}

```

![](../pix/vrmlbar.gif)

```
https://graphics.stanford.edu/courses/cs248-98-fall/Assignments/Assignment3/VRML2_Specification/spec/part1/javascript.html

```

