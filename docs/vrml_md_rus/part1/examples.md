# ![](../pix/vrmllogo2.0.gif)

# The Virtual Reality Modeling Language

# Annex D. Examples

### (informative)

### ISO/IEC DIS 14772-1

#### 4 April 1997

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)D.1 Introduction and  table of contents

This annex provides a variety of VRML examples.

[D.1 Introduction and table of contents](#D.1)

[D.2 Simple example](#D.2)

[D.3 Instancing (sharing)](#D.3)

[D.4 Prototype example](#D.4)

[D.5 Scripting example](#D.5)

[D.6 Geometric properties](#D.6)

[D.7 Prototypes and alternate representations](#D.7)

[D.8 Anchor](#D.8)

[D.9 Directional light](#D.9)

[D.10 PointSet](#D.10)

[D.11 Level of detail](#D.11)

[D.12 Color interpolator](#D.12)

[D.13 TimeSensor](#D.13)

[D.13.1 Click to animate](#D.13.1)

[D.13.2 Alarm clock](#D.13.2)

[D.14 Shuttles and pendulums](#D.14)

[D.15 Robot](#D.15)

[D.16 Chopper](#D.16)

[D.17 Guided tour](#D.17)

[D.18 Elevator](#D.18)

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)D.2 Simple example

This example contains a simple scene defining a view of a red sphere
and a blue box, lit by a directional light:

![](../Images/rsmbb.jpg)

#### Figure D.1: Red sphere meets blue box

```
#VRML V2.0 utf8
Transform {
  children [
    NavigationInfo { headlight FALSE } # We'll add our own light

    DirectionalLight {        # First child
        direction 0 0 -1      # Light illuminating the scene
    }

    Transform {               # Second child - a red sphere
      translation 3 0 1
      children [
        Shape {
          geometry Sphere { radius 2.3 }
          appearance Appearance {
            material Material { diffuseColor 1 0 0 }   # Red
         }
        }
      ]
    }

    Transform {               # Third child - a blue box
      translation -2.4 .2 1
      rotation     0 1 1  .9
      children [
        Shape {
          geometry Box {}
          appearance Appearance {
            material Material { diffuseColor 0 0 1 }  # Blue
         }
        }
      ]
    }

  ] # end of children for world
}

```

[Click here to view this example in a VRML \
browser.](exampleD.2.wrl)

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)D.3 Instancing (sharing)

Reading the following file results in three spheres being drawn. The
first sphere defines a unit sphere at the origin named "Joe",
the second sphere defines a smaller sphere translated along the +x
axis, the third sphere is a reference to the second sphere and is
translated along the -x axis. If any changes occur to the second sphere
(e.g. radius changes), then the third sphere, will change too:

![](../Images/instancing.jpg)

#### Figure D.2: Instancing

```
#VRML V2.0 utf8
Transform {
  children [
    DEF Joe Shape { geometry Sphere {} }
    Transform {
      translation 2 0 0
      children    DEF Joe Shape { geometry Sphere { radius .2 } }
    }
    Transform {
      translation -2 0 0
      children    USE Joe
    }

  ]
}

```

[Click here to view this example in a VRML \
browser.](exampleD.3.wrl) (Note that the spheres are unlit because no appearance was
specified.)

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)D.4 Prototype example

A simple table with variable colours for the legs and top might be
prototyped as:

![](../Images/proto.jpg)

#### Figure D.3: Prototype

```
#VRML V2.0 utf8
PROTO TwoColorTable [ field SFColor legColor  .8 .4 .7
                      field SFColor topColor .6 .6 .1 ]
{
  Transform {
    children [
      Transform {   # table top
       translation 0 0.6 0
        children
          Shape {
            appearance Appearance {
              material Material { diffuseColor IS topColor }
            }
            geometry Box { size 1.2 0.2 1.2 }
          }
      }

      Transform {   # first table leg
       translation -.5 0 -.5
        children
          DEF Leg Shape {
            appearance Appearance {
              material Material { diffuseColor IS legColor }
            }
            geometry Cylinder { height 1 radius .1 }
          }
      }
      Transform {   # another table leg
       translation .5 0 -.5
        children USE Leg
      }
      Transform {   # another table leg
       translation -.5 0 .5
        children USE Leg
      }
      Transform {   # another table leg
       translation .5 0 .5
        children USE Leg
      }
    ] # End of root Transform's children
  } # End of root Transform
} # End of prototype

# The prototype is now defined. Although it contains a
# number of nodes, only the legColor and topColor fields
# are public. Instead of using the default legColor and
# topColor, this instance of the table has red legs and
# a green top:

TwoColorTable {
  legColor 1 0 0 topColor 0 1 0
}
NavigationInfo { type "EXAMINE" }      # Use the Examine viewer

```

[Click here to view this example in a VRML \
browser.](exampleD.4.wrl)

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)D.5 Scripting example

This Script node decides whether or not to open a bank vault given
openVault and combinationEntered messages. To do this, it remembers
whether or not the correct combination has been entered. The Script
node combined with a Sphere, a TouchSensor and a Sound node to show how
is works. When the pointing device is over the sphere, the _combinationEntered_
eventIn of the Script is sent. Then, when the Sphere is touched
(typically when the mouse button is pressed) the Script is sent the _openVault_
eventIn. This generates the _vaultUnlocked_ eventOut which starts
a 'click' sound. Here is the example:

```
#VRML V2.0 utf8

DEF OpenVault Script {
    # Declarations of what's in this Script node:
    eventIn SFTime openVault
    eventIn SFBool combinationEntered
    eventOut SFTime vaultUnlocked
    field SFBool unlocked FALSE

    # Implementation of the logic:
    url "javascript:
        function combinationEntered(value) { unlocked = value; }
        function openVault(value) {
        if (unlocked) vaultUnlocked = value;
    }"
}

Shape {
    appearance Appearance {
        material Material { diffuseColor 1 0 0 }
    }
    geometry Sphere { }
}

Sound {
    source     DEF Click AudioClip {
             url       "click.wav"
             stopTime 1
    }

    minFront   1000
    maxFront   1000
    minBack    1000
    maxBack    1000
}

DEF TS TouchSensor { }

ROUTE TS.isOver TO OpenVault.combinationEntered
ROUTE TS.touchTime TO OpenVault.openVault
ROUTE OpenVault.vaultUnlocked TO Click.startTime

```

Note that the _openVault_ eventIn and the _vaultUnlocked_
eventOut are of type SFTime, which alls them to be wired directly to a
TouchSensor or TimeSensor.

[Click here to view this example in a VRML \
browser.](exampleD.5.wrl)

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)D.6 Geometric properties

The following IndexedFaceSet (contained in a Shape node) uses all four
of the geometric property nodes to specify vertex coordinates, colours
per vertex, normals per vertex, and texture coordinates per vertex
(note that the material sets the overall transparency):

```
#VRML V2.0 utf8

Shape {
    geometry IndexedFaceSet {
        coordIndex [ 0, 1, 3, -1, 0, 2, 3, -1 ]
        coord Coordinate {
            point [ 0 0 0, 1 0 0, 1 0 -1, 0.5 1 0 ]
        }
        color Color {
            color [ 0.2 0.7 0.8, 0.5 0 0, 0.1 0.8 0.1, 0 0 0.7 ]
        }
        normal Normal {
            vector [ 0 0 1, 0 0 1, 0 0 1, 0 0 1 ]
        }
        texCoord TextureCoordinate {
            point [ 0 0, 1 0, 1 0.4, 1 1 ]
        }
    }
    appearance Appearance {
        material Material { transparency 0.5 }
        texture  PixelTexture {
            image 2 2 1 0xFF 0x80 0x80 0xFF
        }
    }
}

```

[Click here to view this example in a VRML \
browser.](exampleD.6.wrl)

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)D.7 Prototypes and  alternate representations

VRML 2.0 has the capability to define new nodes. The following is
an example of a new node RefractiveMaterial. This node behaves as a
Material node with an added field, _indexOfRefraction_. The list
of URLs for the EXTERNPROTO are searched in order. If the browser
recognizes the URN, `urn:inet:foo.com:types:RefractiveMaterial`,
it may treat it as a native type (or load the implementation).
Otherwise, the URL, `http://www.myCompany.com/vrmlNodes/RefractiveMaterial.wrl`,
is used as a backup to ensure that the node is supported on any
browsers. See below for the PROTO implementation that treats
RefractiveMaterial as a Material (and ignores the _refractiveIndex_
field).

```
#VRML V2.0 utf8

# external protype definition
EXTERNPROTO RefractiveMaterial [
    exposedField SFFloat ambientIntensity
    exposedField SFColor diffuseColor
    exposedField SFColor specularColor
    exposedField SFColor emissiveColor
    exposedField SFFloat shininess
    exposedField SFFloat transparency
    exposedField SFFloat indexOfRefraction  ]
[
  "urn:inet:foo.com:types:RefractiveMaterial",
  "http://www.myCompany.com/vrmlNodes/RefractiveMaterial.wrl",
  "refractivematerial.wrl",
]

Shape {
    geometry Sphere { }
    appearance Appearance {
        # Instance of a RefractiveMaterial
        material RefractiveMaterial {
            ambientIntensity  0.2
            diffuseColor      1 0 0
            indexOfRefraction 0.3
        }
    }
}

```

The URL `http://www.myCompany.com/vrmlNodes/RefractiveMaterial.wrl`
contains the following:

```
#VRML V2.0 utf8

PROTO RefractiveMaterial [            # prototype definition
    exposedField SFFloat ambientIntensity  0
    exposedField SFColor diffuseColor      0.5 0.5 0.5
    exposedField SFColor specularColor     0 0 0
    exposedField SFColor emissiveColor     0 0 0
    exposedField SFFloat shininess         0
    exposedField SFFloat transparency      0
    exposedField SFFloat indexOfRefraction 0.1 ]
{
    Material {
        ambientIntensity IS ambientIntensity
        diffuseColor     IS diffuseColor
        specularColor    IS specularColor
        emissiveColor    IS emissiveColor
        shininess        IS shininess
        transparency     IS transparency
    }
}

```

Note that the name of the new node type, **RefractiveMaterial**, is
not used by the browser to decide if the node is native or not; the
URL/URN names determine the node's implementation.

[Click here to view this example in a VRML \
browser.](exampleD.7.wrl)

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)D.8 Anchor

The _target_ parameter can be used by the anchor node to send a
request to load a URL into another frame:

```
Anchor {
  url "http://somehost/somefile.html"
  parameter [ "target=name_of_frame" ]
  children Shape { geometry Cylinder {} }
}

```

An Anchor may be used to bind the viewer to a particular _viewpoint_
in a virtual world by specifying a URL ending with _#viewpointName_,
where _viewpointName_ is the DEF name of a viewpoint defined in
the world. For example:

```
Anchor {
  url "http://www.school.edu/vrml/someScene.wrl#OverView"
  children Shape { geometry Box {} }
}

```

specifies an anchor that puts the viewer in the _someScene_ world
bound to the viewpoint named _OverView_ when the box is chosen
(note that _OverView_ is the DEF name of the viewpoint, not the
value of the viewpoint's description field).

If no world is specified, the current scene is implied. For example:

```
Anchor {
  url "#Doorway"
  children Shape { geometry Sphere {} }
}

```

binds the user's view to the viewpoint with the DEF name _Doorway_
in the current scene.

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)D.9 Directional light

A directional light source illuminates only the objects in its
enclosing grouping node. The light illuminates everything within this
coordinate system including the objects that precede it in the scene
graph as shown below:

```
#VRML V2.0 utf8

Group {
    children [
       DEF UnlitShapeOne Transform {
            translation -3 0 0

           children Shape {
               appearance DEF App Appearance {
                   material Material {
                       diffuseColor 0.8 0.4 0.2
                   }
               }
               geometry Box { }
           }
        }

       DEF LitParent Group {
           children [
               DEF LitShapeOne Transform {
                    translation 0 2 0

                   children Shape {
                       appearance USE App
                       geometry Sphere { }
                   }
               }

               # lights the shapes under LitParent
               DirectionalLight { }
               DEF LitShapeTwo Transform {
                    translation 0 -2 0

                   children Shape {
                       appearance USE App
                       geometry Cylinder { }
                    }
                }
           ]
       }

       DEF UnlitShapeTwo Transform {
            translation 3 0 0

           children Shape {
               appearance USE App
               geometry Cone { }
            }
       }
    ]
}

```

[Click here to view this example in a VRML \
browser.](exampleD.9.wrl)

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)D.10 PointSet

This simple example defines a PointSet composed of 3 points. The first
point is red (1 0 0), the second point is green (0 1 0), and the third
point is blue (0 0 1). The second PointSet instances the Coordinate
node defined in the first PointSet, but defines different colours:

```
#VRML V2.0 utf8

Shape {
    geometry PointSet {
        coord DEF mypts Coordinate {
            point [ 0 0 0, 2 2 2, 3 3 3 ]
        }
        color Color { color [ 1 0 0, 0 1 0, 0 0 1 ] }
    }
}

Transform {
    translation 2 0 0

    children Shape {
        geometry PointSet {
           coord USE mypts
           color Color { color [ .5 .5 0, 0 .5 .5, 1 1 1 ] }
        }
    }
}

```

[Click here to view this example in a VRML \
browser.](exampleD.10.wrl)

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)D.11 Level of detail

The LOD node is typically used for switching between different versions
of geometry at specified distances from the viewer. However, if the
range field is left at its default value, the browser selects the most
appropriate child from the list given. It can make this selection based
on performance or perceived importance of the object. Children should
be listed with most detailed version first just as for the normal case.
This "performance LOD" feature can be combined with the
normal LOD function to give the browser a selection of children from
which to choose at each distance.

In this example, the browser is free to choose either a detailed or a
less-detailed version of the object when the viewer is closer than 10
meters (as measured in the coordinate space of the LOD). The browser
should display the less detailed version of the object if the viewer is
between 10 and 50 meters and should display nothing at all if the
viewer is farther than 50 meters. Browsers should try to honor the
hints given by authors, and authors should try to give browsers as much
freedom as they can to choose levels of detail based on performance.

```
#VRML V2.0 utf8

LOD {
    range [ 10, 50 ]
    level [
        LOD {
            level [
                Shape { geometry Sphere { } }
                DEF LoRes Shape { geometry Box { } }
            ]
        }
        USE LoRes,
        Shape { } # Display nothing
    ]
}

```

For best results, ranges should be specified only where necessary and
LOD nodes should be nested with and without ranges.

[Click here to view this example in a VRML \
browser.](exampleD.11.wrl)

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)D.12 Color interpolator

This example interpolates from red to green to blue in a 10 second
cycle:

```
#VRML V2.0 utf8

DEF myColor ColorInterpolator {
    key        [   0.0,    0.5,    1.0 ]
    keyValue   [ 1 0 0,  0 1 0,  0 0 1 ] # red, green, blue
}

DEF myClock TimeSensor {
    cycleInterval 10.0      # 10 second animation
    loop          TRUE      # infinitely cycling animation
}

Shape {
    appearance Appearance {
        material DEF myMaterial Material { }
    }
    geometry Sphere { }
}

ROUTE myClock.fraction_changed TO myColor.set_fraction
ROUTE myColor.value_changed TO myMaterial.set_diffuseColor

```

[Click here to view this example in a VRML \
browser.](exampleD.12.wrl)

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)D.13 TimeSensor

The TimeSensor is very flexible. The following are some of the many
ways in which it can be used:

1. a TimeSensor can be triggered to run continuously by setting _cycleInterval_
    \> 0, and _loop_ = TRUE, and then routing a time output
    from another node that triggers the loop ( _e. g._, the _touchTime_
    eventOut of a TouchSensor can be routed to the TimeSensor's _startTime_
    to start the TimeSensor running).

2. a TimeSensor can be made to run continuously upon reading by setting _cycleInterval_
    \> 0, _startTime_ \> 0, _stopTime_ = 0, and _loop_
    = TRUE.


### D.13.1 Click to animate

The first example animates a box when the user clicks on it:

```
#VRML V2.0 utf8

DEF XForm Transform {
    children [
        Shape {
            appearance Appearance {
                material Material { diffuseColor 1 0 0 }
            }
            geometry Box {}
        }
        DEF Clicker TouchSensor {}

        # Run once for 2 sec.
        DEF TimeSource TimeSensor { cycleInterval 2.0 }

        # Animate one full turn about Y axis:
        DEF Animation OrientationInterpolator {
            key      [ 0,      .33,       .66,        1.0 ]
            keyValue [ 0 1 0 0, 0 1 0 2.1, 0 1 0 4.2, 0 1 0 0 ]
        }
    ]
}

```

[Click here to view this example in a VRML \
browser.](exampleD.13.1.wrl)

### D.13.2 Alarm clock

The second example plays Westminster Chimes once an hour:

```
#VRML V2.0 utf8

Group {
    children [
        DEF Hour TimeSensor {
            loop          TRUE
            cycleInterval 3600.0 # 60*60 seconds == 1 hour
        }

        Sound {
            source DEF Sounder AudioClip {
                url "http://...../westminster.mid" }
            }
        }
    ]
}

ROUTE Hour.cycleTime TO Sounder.startTime

```

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)D.14 Shuttles and  pendulums

Shuttles and pendulums are great building blocks for composing
interesting animations. This shuttle translates its children back and
forth along the X axis, from -1 to 1 (by default). The _distance_
field can be used to change this default. The pendulum rotates its
children about the Z axis, from 0 to 3.14159 radians and back again (by
default). The _maxAngle_ field can be used to change this default.

```
#VRML V2.0 utf8

PROTO Shuttle [
    field        SFTime  rate      1
    field        SFFloat distance  1
    field        MFNode  children  [ ]
    exposedField SFTime  startTime 0
    exposedField SFTime  stopTime  0
    field        SFBool  loop      TRUE
] {
    DEF F Transform { children IS children }
    DEF T TimeSensor {
        cycleInterval IS rate
        startTime IS startTime
        stopTime IS stopTime
        loop IS loop
    }

    DEF S Script {
        field    SFFloat    distance IS distance
        eventOut MFVec3f    position

        url "javascript:
            function initialize() {
                // constructor:setup interpolator,
                pos1 = new SFVec3f(-distance, 0, 0);
                pos2 = new SFVec3f(distance, 0, 0);
                position = new MFVec3f(pos1, pos2, pos1);
            }",
    }

    DEF I PositionInterpolator {
        key [ 0, 0.5, 1 ]
        keyValue [ -1 0 0, 1 0 0, -1 0 0 ]
    }

    ROUTE T.fraction_changed TO I.set_fraction
    ROUTE I.value_changed TO F.set_translation
    ROUTE S.position TO I.set_keyValue
}

PROTO Pendulum [
    field        SFTime  rate      1
    field        SFFloat maxAngle  3.14159
    field        MFNode  children  [ ]
    exposedField SFTime  startTime 0
    exposedField SFTime  stopTime  0
    field        SFBool  loop      TRUE
] {
    DEF F Transform { children IS children }
    DEF T TimeSensor {
        cycleInterval IS rate
        startTime IS startTime
        stopTime IS stopTime
        loop IS loop
    }
    DEF S Script {
        field    SFFloat    maxAngle IS maxAngle
        eventOut MFRotation rotation

        url "javascript:
            function initialize() {
                // constructor:setup interpolator,
                rot1 = new SFRotation(0, 0, 1, 0);
                rot2 = new SFRotation(0, 0, 1, maxAngle/2);
                rot3 = new SFRotation(0, 0, 1, maxAngle);
                rotation = new MFRotation(rot1, rot2, rot3,
                                          rot2, rot1);
            }",
    }
    DEF I OrientationInterpolator {
        key [ 0, 0.25, 0.5, 0.75, 1 ]
        keyValue [ 0 0 1 0,
                   0 0 1 1.57,
                   0 0 1 3.14,
                   0 0 1 1.57,
                   0 0 1 0 ]
    }

    ROUTE T.fraction_changed TO I.set_fraction
    ROUTE I.value_changed TO F.set_rotation
    ROUTE S.rotation TO I.set_keyValue
}

Transform {
    translation -3 0 0
    children Pendulum {
        rate 3
        maxAngle 6.28
        children Shape { geometry Cylinder { height 5 } }
    }
}

Transform {
    translation 3 0 0
    children Shuttle {
        rate 2
        children Shape { geometry Sphere { } }
    }
}

```

[Click here to view this example in a VRML \
browser.](exampleD.14.wrl)

These nodes can be used to do a continuous animation when _loop_
is TRUE. When _loop_ is FALSE they can perform a single cycle
under control of the _startTime_ and _stopTime_ fields. The _rate_
field controls the speed of the animation. The _children_ field
holds the children to be animated.

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)D.15 Robot

This example is a simple implementation of a robot. This robot has very
simple body parts: a cube for his head, a sphere for his body and
cylinders for arms (he hovers so he has no feet!). He is something of a
sentry--he walks forward and walks back across a path. He does this
whenever the viewer is near. This makes use of the Shuttle and Pendulum
of D.14.

```
#VRML V2.0 utf8

EXTERNPROTO Shuttle [
    field        SFTime  rate
    field        SFFloat distance
    field        MFNode  children
    exposedField SFTime  startTime
    exposedField SFTime  stopTime
    field        SFBool  loop
]
"exampleD.14.wrl#Shuttle"

EXTERNPROTO Pendulum [
    field        SFTime  rate
    field        SFFloat maxAngle
    field        MFNode  children
    exposedField SFTime  startTime
    exposedField SFTime  stopTime
    field        SFBool  loop
]
"exampleD.14.wrl#Pendulum"

Viewpoint {
    position 0 0 150
}

DEF Near ProximitySensor { size 200 200 200 }

DEF Walk Shuttle {
    stopTime 1
    rate 10
    distance 20

    children [
        # The Robot
        Transform {
            rotation 0 1 0 1.57

            children [
                Shape {
                    appearance DEF A Appearance {
                        material Material {
                            diffuseColor 0 0.5 0.7
                        }
                    }
                    geometry Box { } # head
                }
                Transform {
                    scale 1 5 1
                    translation 0 -5 0
                    children Shape {
                        appearance USE A
                        geometry Sphere { }
                    } # body
                }
                Transform {
                    rotation 0 1 0 1.57
                    translation 1.5 0 0

                    children DEF Arm Pendulum {
                        stopTime 1
                        rate 1
                        maxAngle 0.52 # 30 degrees

                        children [
                            Transform {
                                translation 0 -3 0

                                children Shape {
                                    appearance USE A
                                    geometry Cylinder {
                                        height 4
                                        radius 0.5
                                    }
                                }
                            }
                        ]
                    }
                }

                # duplicate arm on other side and flip so
                # it swings in opposition
                Transform {
                    rotation 0 -1 0 1.57
                    translation -1.5 0 0
                    children USE Arm
                }
            ]
        }
    ]
}

ROUTE Near.enterTime TO Walk.startTime
ROUTE Near.enterTime TO Arm.startTime
ROUTE Near.exitTime TO Walk.stopTime
ROUTE Near.exitTime TO Arm.stopTime

```

[Click here to view this example in a VRML \
browser.](exampleD.15.wrl)

Move closer to the robot to start the animation.

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)D.16 Chopper

This example of a helicopter demonstrates how to do simple animation
triggered by a TouchSensor. It uses an EXTERNPROTO to include a Rotor
node from the Internet which does the actual animation.

```
#VRML V2.0 utf8

EXTERNPROTO Rotor [
    field        SFTime  rate
    field        MFNode  children
    exposedField SFTime  startTime
    exposedField SFTime  stopTime
]
"rotor.wrl"

PROTO Chopper [
    field SFTime rotorSpeed 1
] {
    Group {
        children [
            DEF Touch TouchSensor { } # Gotta get touch events
            Inline { url "chopperbody.wrl" }
            DEF Top Rotor {
                # initially, the rotor should not spin
                stopTime 1
                rate IS rotorSpeed
                children Inline { url "chopperrotor.wrl" }
            }
        ]
    }

    DEF RotorScript Script {
        eventIn  SFTime startOrStopEngine
        eventOut SFTime startEngine
        eventOut SFTime stopEngine
        field    SFBool engineStarted FALSE

        url "javascript:
            function startOrStopEngine(value) {
                // start or stop engine:
                if (!engineStarted) {
                    startEngine = value;
                    engineStarted = TRUE;
                }
                else {
                    stopEngine = value;
                    engineStarted = FALSE;
                }
            }"
    }

    ROUTE Touch.touchTime TO RotorScript.startOrStopEngine
    ROUTE RotorScript.startEngine TO Top.startTime
    ROUTE RotorScript.stopEngine TO Top.stopTime
}

Viewpoint { position 0 0 5 }
DEF MyScene Group {
    children DEF MikesChopper Chopper { }
}

```

[Click here to view this example in a VRML \
browser.](exampleD.16.wrl)

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)D.17 Guided tour

VRML обеспечивает управление камерой зрителя с помощью скрипта.
Это полезно для таких объектов, как экскурсии с гидом, карусели
и транспортные средства, такие как автобусы и лифты. В следующих двух
примерах показано несколько способов использования этой функции.

Этот пример представляет собой простую экскурсию по миру с гидом. При входе
перед зрителем появляется шар-путеводитель. Нажмите на него, и начнется экскурсия
по миру. Шар сопровождает пользователя во время его экскурсии.
Датчик приближения гарантирует, что тур начнется только в том случае, если пользователь находится
близко к начальной точке отсчета. Обратите внимание, что это делается без
использования скриптов благодаря _touchTime_, выводимому сенсорным датчиком.

```
#VRML V2.0 utf8

Group {
    children [
        Transform {
            translation 0 -1 0

            children Shape {
                appearance Appearance {
                    material Material { }
                }
                geometry Box { size 30 0.2 30 }
            }
        }
        Transform {
            translation -1 0 0

            children Shape {
                appearance Appearance {
                    material Material {
                        diffuseColor 0.5 0.8 0
                    }
                }
                geometry Cone { }
            }
        }
        Transform {
            translation 1 0 0

            children Shape {
                appearance Appearance {
                    material Material {
                        diffuseColor 0 0.2 0.7
                    }
                }
                geometry Cylinder { }
            }
        }

        DEF GuideTransform Transform {
            children [
                DEF TourGuide Viewpoint { jump FALSE },
                DEF ProxSensor ProximitySensor { size 50 50 50 }
                DEF StartTour TouchSensor { },
                Transform {
                    translation 0.6 0.4 8

                    children Shape {
                        appearance Appearance {
                            material Material {
                                diffuseColor 1 0.6 0
                            }
                        }
                        geometry Sphere { radius 0.2 }
                    } # the guide orb
                }
            ]
        }
    ]
}

DEF GuidePI PositionInterpolator {
    key [ 0, 0.2, 0.3, 0.5, 0.6, 0.8, 0.9, 1 ]
    keyValue [ 0 0 0, 0 0 -5,
               2 0 -5, 2 6 -15
               -4 6 -15, -4 0 -5,
               0 0 -5, 0 0 0
    ]
}

DEF GuideRI OrientationInterpolator {
    key [ 0, 0.2, 0.3, 0.5, 0.6, 0.8, 0.9, 1 ]
    keyValue [ 0 1 0 0, 0 1 0 0,
               0 1 0 1.2, 0 1 0 3,
               0 1 0 3.5, 0 1 0 5,
               0 1 0 0, 0 1 0 0,
    ]
}

DEF TS TimeSensor { cycleInterval 30 } # 60 second tour

ROUTE ProxSensor.isActive TO StartTour.set_enabled
ROUTE StartTour.touchTime TO TS.startTime
ROUTE TS.isActive TO TourGuide.set_bind
ROUTE TS.fraction_changed TO GuidePI.set_fraction
ROUTE TS.fraction_changed TO GuideRI.set_fraction
ROUTE GuidePI.value_changed TO GuideTransform.set_translation
ROUTE GuideRI.value_changed TO GuideTransform.set_rotation

```

[Click here to view this example in a VRML \
browser.](exampleD.17.wrl)

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif)D.18 Elevator

Это еще один пример анимации камеры путем изображения
лифт для облегчения доступа в многоэтажное здание. Для этого примера
Показано 2-этажное здание, предполагается, что лифт есть.
уже на первом этаже. Чтобы подняться, пользователь просто встает на
лифтовая платформа. Датчик приближения срабатывает и запускает лифт вверх.
автоматически. Дополнительные функции, такие как кнопки вызова за пределами
Лифт, двери лифта и кнопки выбора этажа могут быть добавлены к
сделать лифт более удобным в использовании.

```
#VRML V2.0 utf8

Transform {
    translation 0 0 -3.5

    children Shape {
        appearance Appearance {
            material Material {
                diffuseColor 0 1 0
            }
        }
        geometry Cone { }
    }
}

Transform {
    translation 0 4 -3.5

    children Shape {
        appearance Appearance {
            material Material {
                diffuseColor 1 0 0
            }
        }
        geometry Cone { }
    }
}

Transform {
    translation 0 8 -3.5

    children Shape {
        appearance Appearance {
            material Material {
                diffuseColor 0 0 1
            }
        }
        geometry Cone { }
    }
}

Group {
    children [
        DEF ETransform Transform {
            children [
                DEF EViewpoint Viewpoint { jump FALSE }
                DEF EProximity ProximitySensor { size 2 5 5 }
                Transform {
                    translation 0 -1 0

                    children Shape {
                        appearance Appearance {
                            material Material { }
                        }
                        geometry Box { size 2 0.2 5 }
                    }
                }
            ]
        }
    ]
}

DEF ElevatorPI PositionInterpolator {
    key [ 0, 1 ]
    keyValue [ 0 0 0, 0 8 0 ] # a floor is 4 meters high
}
DEF TS TimeSensor { cycleInterval 10 } # 10 second travel time

ROUTE EProximity.enterTime TO TS.startTime
ROUTE TS.isActive TO EViewpoint.set_bind
ROUTE TS.fraction_changed TO ElevatorPI.set_fraction
ROUTE ElevatorPI.value_changed TO ETransform.set_translation

```

[Click here to view this example in a VRML \
browser.](exampleD.18.wrl)

![](../pix/vrmlbar.gif)

```
https://graphics.stanford.edu/courses/cs248-98-fall/Assignments/Assignment3/VRML2_Specification/spec/part1/examples.html

```

