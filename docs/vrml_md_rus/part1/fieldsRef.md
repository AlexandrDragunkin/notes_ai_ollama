# ![](../pix/vrmllogo2.0.gif)

# The Virtual Reality Modeling Language

# 5\. Field and Event Reference

### ISO/IEC DIS 14772-1

#### 4 April 1997

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif) 5.1 Introduction

### 5.1.1 Table of Contents

5.1 [Introduction](#Introduction)

       5.1.1 [Table of \
contents](#5.1.1)

       5.1.2 [Description](#5.1.2)

5.2 [SFBool](#SFBool)

5.3 [SFColor and MFColor](#SFColor)

5.4 [SFFloat and MFFloat](#SFFloat)

5.5 [SFImage](#SFImage)

5.6 [SFInt32 and MFInt32](#SFInt32)

5.7 [SFNode and MFNode](#SFNode)

5.8 [SFRotation and MFRotation](#SFRotation)

5.9 [SFString and MFString](#SFString)

5.10 [SFTime](#SFTime)

5.11 [SFVec2f and MFVec2f](#SFVec2f)

5.12 [SFVec3f and MFVec3f](#SFVec3f)

### 5.1.2 Description

This clause describes the syntax and general semantics of _fields_
and _events,_ the elemental data types used by VRML nodes to
define objects (see [Clause 6. "Node \
Reference"](nodesRef.html)). Nodes are composed of fields and events (see [Clause 4. "Concepts"](concepts.html)). The types
defined in this annex are used by both fields and events.

There are two general classes of fields and events: fields/events that
contain a single value (where a value may be a single number, a vector,
or even an image), and fields/events that contain an ordered list of
multiple values. Single-valued fields/events have names that begin with **`SF. `** Multiple-valued
fields/events have names that begin with **`MF`**.

Multiple-valued fields/events are written as an ordered list of values
enclosed in square brackets and separated by whitespace (e.g., by
commas). If the field or event has zero values, only the square
brackets ("\[ \]") are written. The last value may optionally
be followed by whitespace (e.g., commas or spaces). If the field
has exactly one value, the brackets may be omitted. For example, all of
the following are valid for a multiple-valued MFInt32 field named _foo_
containing the single integer value 1:

```
    foo 1
    foo [1,]
    foo [ 1 ]

```

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)5.2  SFBool

SFBool is a field or event containing a single boolean value. SFBools
are written as TRUE or FALSE. For example,

```
    fooBool FALSE

```

is an SFBool field, _fooBool_, defining a FALSE value.

The initial value of an SFBool eventOut is FALSE.

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)5.3 SFColor and MFColor

SFColor specifies one RGB (red-green-blue) colour triple. MFColor
specifies zero or more RGB triples. Each colour is written to the VRML
file as an RGB triple of floating point numbers in ISO C floating point
format (see [2.\[ISOC\]](references.html#[ISOC])) in the range
0.0 to 1.0. For example:

```
    fooColor [ 1.0 0. 0.0, 0 1 0, 0 0 1 ]

```

is an MFColor field, _fooColor_, containing the three primary
colours red, green, and blue.

The initial value of an SFColor eventOut is (0 0 0). The initial value
of an MFColor eventOut is \[ \].

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)5.4 SFFloat and MFFloat

SFFloat specifies one single-precision floating point number. MFFloat
specifies zero or more single-precision floating point numbers.
SFFloats and MFFloats are written to the VRML file in ISO C
floating point format (see [2.\[ISOC\]](references.html#[ISOC])).
For example:

```
    fooFloat [ 3.1415926, 12.5e-3, .0001 ]

```

is an MFFloat field, _fooFloat_, containing three floating point
values.

The initial value of an SFFloat eventOut is 0.0. The initial value of
an MFFloat eventOut is \[ \].

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)5.5 SFImage

The SFImage field or event defines a single uncompressed 2-dimensional
pixel image. SFImage fields and events are written to the VRML file as
three integers representing the width, height and number of components
in the image, followed by width\*height hexadecimal or integer values
representing the pixels in the image, separated by whitespace:

```
    fooImage <width> <height> <num components> <pixels values>

```

Pixel values are limited to 256 levels of intensity (i.e., 0-255
decimal or 0x00-0xFF hexadecimal). A one-component image specifies
one-byte hexadecimal or integer values representing the intensity of
the image. For example, `0xFF` is full intensity in hexadecimal
(255 in decimal), `0x00` is no intensity (0 in decimal). A
two-component image specifies the intensity in the first (high) byte
and the alpha opacity in the second (low) byte. Pixels in a
three-component image specify the red component in the first (high)
byte, followed by the green and blue components (e.g., `0xFF0000`
is red, ` 0x00FF00` is green, `0x0000FF` is blue).
Four-component images specify the alpha opacity byte after
red/green/blue (e.g., `0x0000FF80` is semi-transparent
blue). A value of `0x00` is completely transparent, 0xFF is
completely opaque. Note that alpha equals (1.0 - transparency),
if alpha and transparency range from 0.0 to 1.0.

Each pixel is read as a single unsigned number. For example, a
3-component pixel with value `0x0000FF` may also be written as `0xFF`
(hexadecimal) or `255 `(decimal). Pixels are specified from
left to right, bottom to top. The first hexadecimal value is the lower
left pixel and the last value is the upper right pixel.

For example,

```
    fooImage 1 2 1 0xFF 0x00

```

is a 1 pixel wide by 2 pixel high one-component (i.e., greyscale)
image, with the bottom pixel white and the top pixel black. As another
example,

```
    fooImage 2 4 3 0xFF0000 0xFF00 0 0 0 0 0xFFFFFF 0xFFFF00
                   # red    green  black.. white    yellow

```

is a 2 pixel wide by 4 pixel high RGB image, with the bottom left pixel
red, the bottom right pixel green, the two middle rows of pixels black,
the top left pixel white, and the top right pixel yellow.

The initial value of an SFImage eventOut is (0 0 0).

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)5.6 SFInt32 and MFInt32

The SFInt32 field and event specifies one 32-bit integer. The MFInt32
field and event specifies zero or more 32-bit integers. SFInt32 and
MFInt32 fields and events are written to the VRML file as an integer in
decimal or hexadecimal (beginning with '0x') format. For example:

```
    fooInt32 [ 17, -0xE20, -518820 ]

```

is an MFInt32 field containing three values.

The initial value of an SFInt32 eventOut is 0. The initial value of an
MFInt32 eventOut is \[ \].

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)5.7 SFNode and MFNode

The SFNode field and event specifies a VRML node. The MFNode field and
event specifies zero or more nodes. The following example illustrates
valid syntax for an MFNode field, _fooNode_, defining four nodes:

```
    fooNode [ Transform { translation 1 0 0 }
              DEF CUBE Box { }
              USE CUBE
              USE SOME_OTHER_NODE  ]

```

The SFNode and MFNode fields and events may contain the keyword NULL to
indicate that it is empty.

The initial value of an SFNode eventOut is NULL. The initial value of
an MFNode eventOut is \[ \].

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)5.8 SFRotation and MFRotation

The SFRotation field and event specifies one arbitrary rotation. The
MFRotation field and event specifies zero or more arbitrary rotations.
An SFRotation is written to the VRML file as four ISO C
floating point values (see [2.\[ISOC\]](references.html#[ISOC]))
separated by whitespace. The first three values specify a normalized
rotation axis vector about which the rotation takes place. The fourth
value specifies the amount of right-handed rotation about that axis in
radians. An MFRotation specifies 0 or more rotations. For example, an
SFRotation containing a PI radians rotation about the Y axis is:

```
    fooRot 0.0 1.0 0.0 3.14159265

```

The 3x3 matrix represention of a rotation (x y z a) is

```
    [ tx2+c    txy+sz    txz-sy
      txy-sz   ty2+c     tyz+sx
      txz+sy   tyz-sx    tz2+c  ]

    where c = cos(a), s = sin(a), and t = 1-c

```

The initial value of an SFRotation eventOut is (0 0 1 0). The initial
value of an MFRotation eventOut is \[ \].

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)5.9 SFString and MFString

The SFString and MFString fields and events contain strings formatted
with the UTF-8 universal character set (see [2.\[UTF8\]](references.html#[UTF8])). SFString specifies a
single string. The MFString specifies zero or more strings. Strings are
written to the VRML file as a sequence of UTF-8 octets enclosed in
double quotes (e.g., `"string"`).

Any characters (including LineFeeds and '#') may appear within the
quotes. A double quote character within the string is preceded with a
backslash. A backslash character within the string is also preceded
with a backslash forming two backslashes. For example:

```
    fooString [ "One, Two, Three", "He said, \"Immel did it!\"" ]

```

is an MFString field, _fooString_, with two valid strings.

The initial value of an SFString eventOut is "" (the empty
string). The initial value of an MFString eventOut is \[ \].

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)5.10 SFTime

The SFTIme field and event specifies a single time value. Time values
are written to the VRML file as a double-precision floating point
number in ISO C floating point format (see [2.\[ISOC\]](references.html#[ISOC])). Time values are specified
as the number of seconds from a specific time origin. Typically, SFTime
fields and events represent the number of seconds since Jan 1, 1970,
00:00:00 GMT.

The initial value of an SFTime eventOut is -1. The initial value of an
MFTime eventOut is \[ \].

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)5.11 SFVec2f and MFVec2f

An SFVec2f field or event specifies a two-dimensional (2D) vector. An
MFVec2f field or event specifies zero or more 2D vectors. SFVec2f's and
MFVec2f's are written to the VRML file as a pair of ISO C
floating point values (see [2.\[ISOC\]](references.html#[ISOC]))
separated by whitespace. For example:

```
    fooVec2f [ 42 666, 7 94 ]

```

is an MFVec2f field, _fooVec2f_, with two valid vectors.

The initial value of an SFVec2f eventOut is (0 0). The initial value of
an MFVec2f eventOut is \[ \].

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)5.12 SFVec3f and MFVec3f

An SFVec3f field or event specifies a three-dimensional (3D) vector. An
MFVec3f field or event specifies zero or more 3D vectors. SFVec3f's and
MFVec3f's are written to the VRML file as three ISO C floating
point values (see [2.\[ISOC\]](references.html#[ISOC]))
separated by whitespace. For example:

```
    fooVec3f [ 1 42 666, 7 94 0 ]

```

is an MFVec3f field, _fooVec3f_, with two valid vectors.

The initial value of an SFVec3f eventOut is (0 0 0). The initial value
of an MFVec3f eventOut is \[ \].

![](../pix/vrmlbar.gif)

```
https://graphics.stanford.edu/courses/cs248-98-fall/Assignments/Assignment3/VRML2_Specification/spec/part1/fieldsRef.html

```

