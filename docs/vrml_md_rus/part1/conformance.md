# ![](../pix/vrmllogo2.0.gif)

# The Virtual Reality Modeling Language

# 7\. Conformance and minimum support requirements

### ISO/IEC DIS 14772-1

#### 4 April 1997

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif) 7.1 Introduction

### 7.1.1 Table of contents

7.1 [Introduction](#7.1)

      7.1.1 [Table of contents](#7.1.1)

      7.1.2 [Objectives](#7.1.2)

      7.1.3 [Scope](#7.1.3)

7.2 [Conformance](#7.2)

      7.2.1 [Conformance of \
VRML files](#7.2.1)

      7.2.2 [Conformance of \
VRML generators](#7.2.2)

      7.2.3 [Conformance of \
VRML browsers](#7.2.3)

7.3 [Minimum support requirements](#MinimumSupportRequirements)

      7.3.1 [Minimum support \
requirements for generators](#7.3.1)

      7.3.2 [Minimum support \
requirements for browsers](#7.3.2)

      7.3.3 [VRML requirements \
for conforming to the base profile](#7.3.3)

      7.3.4 [Sound priority, \
attenuation, and spatialization](#7.3.4)

### 7.1.2 Objectives

This clause addresses conformance of VRML files, VRML generators and
VRML browsers.

The primary objectives of the specifications in this clause are:

1. to promote interoperability by eliminating arbitrary subsets of, or
    extensions to, ISO/IEC 14772;

2. to promote uniformity in the development of conformance tests;

3. to promote consistent results across VRML browsers;

4. to facilitate automated test generation.


### 7.1.3 Scope

Conformance is defined for VRML files and for VRML browsers. For VRML
generators, conformance guidelines are presented for enhancing the
likelihood of successful interoperability.

A concept of _base profile conformance_ is defined to ensure
interoperability of VRML generators and VRML browsers. Base profile
conformance is based on a set of limits and minimal requirements. Base
profile conformance is intended to provide a functional level of
reasonable utility for VRML generators while limiting the complexity
and resource requirements of VRML browsers. Base profile conformance
may not be adequate for all uses of VRML.

This clause addresses the VRML data stream and implementation
requirements. Implementation requirements include the latitude allowed
for VRML generators and VRML browsers. This clause does not
directly address the environmental, performance, or resource
requirements of the generator or browser.

This clause does not define the application requirements or dictate
application functional content within a VRML file.

The scope of this clause is limited to rules for the open interchange
of VRML content.

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif) 7.2 Conformance

### 7.2.1 Conformance of VRML  files

A VRML file is _syntactically correct_ according to ISO/IEC
14772 if the following conditions are met:

1. The VRML file contains as its first element a VRML header
    comment (see 4.4.1)

2. All entities contained therein match the functional specification of
    the corresponding entities of ISO/IEC 14772-1. The VRML file
    shall obey the relationships defined in the formal grammar and all
    other syntactic requirements.

3. The sequence of entities in the VRML file obeys the relationships
    specified in ISO/IEC 14772-1 producing the structure specified in
    ISO/IEC 14772-1.

4. All field values in the VRML file obey the relationships
    specified in ISO/IEC 14772-1 producing the structure specified in
    ISO/IEC 14772-1.

5. No nodes appear in the VRML file other than those specified in
    ISO/IEC 14772-1 unless required for the encoding technique or those
    defined by the PROTO or EXTERNPROTO entities.

6. The VRML file is encoded according to the rules of ISO/IEC 14772.


A VRML file conforms to the _base profile_ if:

7. it is syntactically correct;

8. it meets the restrictions of Table 7-1.


### 7.2.2 Conformance of VRML generators

Unlike VRML files and VRML browsers, conformance of VRML generators is
not specifically defined. However, the probability of successful
interoperability of a VRML generator with VRML browsers conforming to
the base profile (see " [7.2.3 Conformance of VRML \
browsers](#7.2.3)") is significantly improved if the generator is
capable of operating in a mode that generates VRML files conforming to
the base profile.

### 7.2.3 Conformance of VRML browsers

A VRML browser conforms to the base profile if:

1. it is able to read any VRML file that conforms to the base profile;

2. it presents the graphical and audio characteristics of the VRML nodes
    in any VRML file that conforms to the base profile, within the latitude
    defined in this clause;

3. it correctly handles user interaction and generation of events as
    specified in ISO/IEC 14772, within the latitude defined in this clause;

4. it satisfies the requirements of "7.3.2 Minimum support
    requirements for browsers" and as enumerated in Table 7-1.


![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif) 7.3 Minimum  support requirements

### 7.3.1 Minimum support requirements for generators

There is no minimum complexity which is required of (or appropriate
for) VRML generators. Any compliant set of nodes of arbitrary
complexity may be generated, as appropriate to represent application
content.

### 7.3.2 Minimum support requirements for browsers

This subclause defines the minimum complexity which shall be supported
by a VRML browser. Browser implementations may choose to support
greater limits but may not reduce the limits described in Table 7-1.
When the VRML file contains nodes which exceed the limits implemented
by the browser, the results are undefined. Where latitude is specified
in Table 7-1 for a particular node, full support is required for other
aspects of that node.

### 7.3.3 VRML requirements for conforming to the base  profile

In the following table, the first column defines the item for which
conformance is being defined. In some cases, general limits are defined
but are later overridden in specific cases by more restrictive limits.
The second column defines the requirements for a VRML file conforming
to the base profile; if a file contains any items that exceed these
limits, it may not be possible for a VRML browser conforming to the
base profile to successfully parse that file. The third column defines
the minimum complexity for a VRML scene that a VRML browser conforming
to the base profile shall be able to present to the user. The word
"ignore" in the minimum browser support column refers only to
the display of the item; in particular, set\_ events to ignored
exposedFields must still generate corresponding _\_changed_ events.

### Table 7-1: Specifications for VRML browsers conforming to the base   profile

**Item**

**(node/field/statement)****File Limit****Minimum Browser Support**
 All groups 500 children. 500 children. Ignore _bboxCenter_ and _bboxSize_. All interpolators 1000 key-value pairs. 1000 key-value pairs. All lights 8 simultaneous lights. 8 simultaneous lights. Names for DEF/PROTO/field 50 utf8 octets. 50 utf8 octets. All _url_ fields 10 URLs. 10 URLs. URN's ignored.

 Support http, file, and ftp protocols.

 Support relative URLs where relevant.PROTO/EXTERNPROTO 30 fields, 30 eventIns, 30 eventOuts, 30 exposedFields. 30 fields, 30 eventIns, 30 eventOuts, 30 exposedFields. PROTO definition nesting depth 5 levels. 5 levels. SFBool No restrictions. Full support. SFColor No restrictions. Full support. SFFloat No restrictions. Full support. SFImage 256 width. 256 height. 256 width. 256 height. SFInt32 No restrictions. Full support. SFNode No restrictions. Full support. SFRotation No restrictions. Full support. SFString 30,000 utf8 octets. 30,000 utf8 octets. SFTime No restrictions. Full support. SFVec2f 15,000 values. 15,000 values. SFVec3f 15,000 values. 15,000 values. MFColor 15,000 values. 15,000 values. MFFloat 1,000 values. 1,000 values. MFInt32 20,000 values. 20,000 values. MFNode 500 values. 500 values. MFRotation 1,000 values. 1,000 values. MFString 30,000 utf8 octets per string, 10 strings. 30,000 utf8 octets per string, 10 strings. MFTime 1,000 values. 1,000 values. MFVec2f 15,000 values. 15,000 values. MFVec3f 15,000 values. 15,000 values. Anchor No restrictions. Ignore _parameter_. Ignore _description._Appearance No restrictions. Full support. AudioClip 30 second uncompressed PCM WAV. 30 second uncompressed PCM WAV. Ignore _description_. Background No restrictions. One _skyColor_, one _groundColor_, panorama
 images as per ImageTexture. Billboard Restrictions as for all groups. Full support except as for all groups. Box No restrictions. Full support. Collision Restrictions as for all groups. Full support except as for all groups. Any navigation
 behaviour acceptable when collision occurs. Color 15,000 colours. 15,000 colours. ColorInterpolator Restrictions as for all interpolators. Full support except as for all interpolators. Cone No restrictions. Full support. Coordinate 15,000 points. 15,000 points. CoordinateInterpolator 15,000 coordinates per _keyValue_. Restrictions as for
 all interpolators. 15,000 coordinates per _keyValue_. Support as for all
 interpolators. Cylinder No restrictions. Full support. CylinderSensor No restrictions. Full support. DirectionalLight No restrictions. Not scoped by parent Group or Transform.ElevationGrid 16,000 heights. 16,000 heights. Extrusion (# _crossSection_ points)\*(# _spine_ points) <=
 2,500. (# _crossSection_ points)\*(# _spine_ points) <=
 2,500. Fog No restrictions. "EXPONENTIAL" treated as "LINEAR" FontStyle No restrictions. If the values of the text aspects character set, _family_, _style_
 cannot be simultaneously supported, the order of precedence shall be:
 1) character set 2) _family_ 3) _style_. Browser
 must display all characters in ISO 8859-1 character set. Group Restrictions as for all groups. Full support except as for all groups. ImageTexture JPEG and PNG format. Restrictions as for PixelTexture. JPEG and PNG format. Support as for PixelTexture. IndexedFaceSet 10 vertices per face. 5000 faces. Less than 15,000 indices. 10 vertices per face. 5000 faces. 15,000 indices in any
 index field. IndexedLineSet 15,000 total vertices. 15,000 indices in any index field. 15,000 total vertices. 15,000 indices in any index field. Inline No restrictions. Full support except as for all groups. LOD Restrictions as for all groups. At least first 4 _level_/ _range_ combinations
 interpreted, and support as for all groups. Implementations may
 disregard _level_ distances. Material No restrictions. Ignore ambient intensity. Ignore specular colour. Ignore
 emissive colour. One-bit transparency; transparency values >= 0.5
 transparent. MovieTexture MPEG1-Systems and MPEG1-Video formats. MPEG1-Systems and MPEG1-Video formats. Display one active
 movie texture. Ignore _speed_ field. NavigationInfo No restrictions. Ignore _avatarSize_. Ignore _visibilityLimit_. Normal 15,000 normals 15,000 normals NormalInterpolator 15,000 normals per _keyValue_. Restrictions as for all
 interpolators. 15,000 normals per _keyValue_. Support as for all
 interpolators. OrientationInterpolator Restrictions as for all interpolators. Full support except as for all interpolators. PixelTexture 256 width. 256 height. 256 width. 256 height. Display fully transparent and fully
 opaque pixels. PlaneSensor No restrictions. Full support. PointLight No restrictions. Ignore _radius_. Linear attenuation. PointSet 5000 points. 5000 points. PositionInterpolator Restrictions as for all interpolators. Full support except as for all interpolators. ProximitySensor No restrictions. Full support. ScalarInterpolator Restrictions as for all interpolators. Full support except as for all interpolators. Script 25 eventIns. 25 eventOuts. 25 fields. 25 eventIns. 25 eventOuts. 25 fields. Shape No restrictions. Full support. Sound No restrictions. 2 active sounds. Linear distance attenuation. No
 spatialization. See 7.3.4.Sphere No restrictions. Full support.SphereSensor No restrictions. Full support.SpotLight No restriction Ignore _beamWidth_. Ignore _radius_. Linear
 attenuation.Switch Restrictions as for all groups. Full support except as for all groups.Text 100 characters per string. 100 strings. 100 characters per string. 100 strings.TextureCoordinate 15,000 coordinates. 15,000 coordinates.TextureTransform No restrictions. Full support.TimeSensor No restrictions. Ignored if _cycleInterval_ < 0.01 second.TouchSensor No restrictions. Full support.Transform Restrictions as for all groups. Full support except as for all groups.Viewpoint No restrictions. Ignore _fieldOfView_. Ignore _description_.VisibilitySensor No restrictions. Always visible. WorldInfo No restrictions. Ignored.

### 7.3.4 Sound priority, attenuation, and spatialization

#### 7.3.4.1 Sound priority

If the browser does not have the resources to play all of the currently
active sounds, it is recommended that the browser sort the active
sounds into an ordered list using the following sort keys in the order
specified:

1. Decreasing _priority;_
2. For sounds with _priority_ \> 0.5, increasing (now- _startTime_);

3. Decreasing _intensity_ at viewer location ( _intensity_ \\*
    intensity attenuation);


where _priority_ is the _priority_ field of the Sound node,
now represents the current time, _startTime_ is the _startTime_
field of the audio source node specified in the _source_ field,
and intensity attenuation refers to the intensity multiplier derived
from the linear decibel attenuation ramp between inner and outer
ellipsoids.

It is important that sort key 2 be used for the high priority (event
and cue) sounds so that new cues will be heard even when the browser is
"full" of currently active high priority sounds. Sort key 2
should not be used for normal priority sounds, so selection among them
will be based on sort key 3 (intensity at the location of the viewer).

The browser shall play as many sounds from the beginning of this sorted
list as it can given available resources and allowable latency between
rendering. On most systems, the resources available for MIDI streams
are different from those for playing sampled sounds, thus it may be
beneficial to maintain a separate list to handle MIDI data.

#### 7.3.4.2 Sound attenuation and spatialization

In order to create a linear decrease in loudness as the viewer moves
from the inner to the outer ellipsoid of the sound, the attenuation
must be based on a linear decibel ramp. To make the falloff consistent
across browsers, the decibel ramp is to vary from 0 dB at the minimum
ellipsoid to -20 dB at the outer ellipsoid. Sound nodes with an outer
ellipsoid that is ten times larger than the minimum will display the
inverse square intensity dropoff that approximates sound attenuation in
an anechoic environment.

Browsers may support spatial localization of sounds whose _spatialize_
field is TRUE as well as their underlying sound libraries will allow.
Browsers shall at least support stereo panning of non-MIDI sounds based
on the angle between the viewer and the source. This angle is obtained
by projecting the Sound _location_ (in global space) onto the XZ
plane of the viewer. Determine the angle between the Z-axis and the
vector from the viewer to the transformed _location_, and assign a
pan value in the range \[0.0, 1.0\] as depicted in Figure 7.1. Given this
pan value, left and right channel levels can be obtained using the
following equations:

```
    leftPanFactor  = 1 - pan2

    rightPanFactor = 1 - (1 - pan)2

```

![](../Images/Sound2.gif)

#### Figure 7.1: Stereo Panning

Using this technique, the loudness of the sound is modified by the _intensity_
field value, then distance attenuation to obtain the unspatialized
audio output. The values in the unspatialized audio output are then
scaled by leftPanFactor and rightPanFactor to determine the final left
and right output signals. The use of more sophisticated localization
techniques is encouraged, but not required (see [E.\[SNDB\]](bibliography.html#[SNDB])).

![](../pix/vrmlbar.gif)

```
https://graphics.stanford.edu/courses/cs248-98-fall/Assignments/Assignment3/VRML2_Specification/spec/part1/conformance.html

```

