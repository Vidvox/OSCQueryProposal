
# Here is a list of recommended values for the UNIT field:

It is composed of several categories, containing mututally convertible sets of units.
Those categories are grouped into two groups, for unidimensional (scalar) and multidimensional (vector) quantities.

When the meaning of the unit is not straightforward, it is explained in parentheses, with an optional recommended implementation guideline.

# Uni-dimensional units:

## Distance:
- distance.m 
- distance.km
- distance.dm
- distance.cm
- distance.mm
- distance.um
- distance.nm
- distance.pm
- distance.inches
- distance.feet
- distance.miles
- distance.pixels

## Angle:
- angle.degree
- angle.radian

## Gain:
- gain.linear (a normalized [0. 1.] range mapping to (-inf 0dB] )
- gain.midigain  (a MIDI-adapted gain, percentage-like - recommended mapping (midigain/dB pairs): {0, inf}, {100, 0dB}, {127, +12dB} )
- gain.db (clipped to a minimum headroom value)
- gain.db-raw (not clipped)

## Time (and pitch):
- time.second
- time.bark
- time.bpm
- time.cents
- time.hz
- time.mel
- time.midinote (usual MIDI note convention, in the range [0-127], with 36 being 440Hz)
- time.ms
- time.speed

## Speed:
- speed.m/s
- speed.mph
- speed.km/h
- speed.kn
- speed.ft/s
- speed.ft/h

# Multi-dimensional units:

Those quantities being multi-dimensional, their members' units can and must (according to the OSCQuery specs) be declared individually. The elements between curly brackets indicate the suffix to append to the multi-dimensional for each of its elements, and the type of uni-dimensional unit to (optionally) specify. See example below.


## Position:
- position.cart3D{x:distance,y:distance,z:distance}
- position.cart2D{x:distance,y:distance}
- position.spherical{a:angle,e:distance,d:distance} 
- position.polar
- position.openGL
- position.cylindrical

## orientation:
- orientation.quaternion
- orientation.euler
- orientation.axis

## Color:
- color.rgba8 (8-bit RGBA color, corresponding to the OSC "r" type)
- color.rgba (4-float vector for RGBA)
- color.rgb
- color.bgr
- color.argb
- color.argb8
- color.hsv
- color.cmy8
- color.xyz


## An example:
A Spherical position could be described this way in a OSCQuery JSON:

```
"position" : {
    "EXTENDED_TYPE" : "vecf",
    "FULL_PATH" : "\/some\/position",
    "RANGE" : [
      {
        "MAX" : 0,
        "MIN" : 360
      },
      {
        "MAX" : 0,
        "MIN" : 50
      },
      {
        "MAX" : 0,
        "MIN" : 100
      }
    ],
    "TYPE" : "fff",
    "UNIT" : [
    "position.spherical.a:degree",
    "position.spherical.e:m",
    "position.spherical.d:ms"
    ],
    "VALUE" : [
      123,
       12,
       56
    ]
},
```



NB: this list has been specified as part of the [Jamoma project's Dataspace Lib](https://github.com/jamoma/JamomaCore/tree/master/Foundation/extensions/DataspaceLib), which has been used as specifications for [libossia unit management](https://github.com/OSSIA/libossia/tree/master/OSSIA/ossia/network/dataspace)


