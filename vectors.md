# Multi-dimensional quantities (vectors):


Similarly (and complementary) to **UNIT**s for scalars (aka single-dimension quantities), vectors (aka multi-dimensional quantities) can be classified into sets of mututally convertible values.


Those quantities being multi-dimensional, their members' units can and must (according to the OSCQuery specs) be declared individually. The elements between curly brackets indicate pairs of each member's name, and the  **UNIT** category to which it belongs .



## Position: (2 or 3 dimensions)
- position.cart2D {x:distance, y:distance}
- position.polar {a:angle, d:distance} 
- position.cart3D {x:distance, y:distance, z:distance}
- position.spherical {a:angle, e:distance, d:distance} 
- position.cylindrical {d:distance, a:angle, z:distance}
- position.openGL {x:distance, y:distance, z:distance} (inverted Z and Y axis, compared to cart3D)


## orientation: (3 or 4 dimensions)
- orientation.quaternion {a:distance, b:distance, c:distance, d:distance} ( extension of the complex numbers for 3D orientation, in the form a+bi+cj+dk)
- orientation.euler {y:angle, p:angle, r:angle} (yaw, pitch, roll: describing the orientation of a rigid body with respect to a fixed coordinate system)
- orientation.axis {x: distance, y:distance, z:distance, w:angle} (An angle (w) relative to a 3-dimensional axis of vector XYZ)

## Color:
- color.rgba8 (8-bit RGBA color, corresponding to the OSC "r" type)
- color.rgba (4-float vector for RGBA)
- color.rgb
- color.bgr
- color.argb
- color.argb8
- color.hsv
- color.cmy8 (8-bit cyan, magenta, yellow color)
- color.xyz


## An example:
A Spherical position could be described this way in a OSCQuery JSON:

``
"position" : {
    "EXTENDED_TYPE" : "position.spherical",
    "FULL_PATH" : "\/some\/position",
    "RANGE" : [
       [
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
       ]
    ],
    "TYPE" : "[fff]",
    "UNIT" : [
       [
        "angle.degree",
        "distance.m",
        "distance.m"
       ],
     ]
    "VALUE" : [
      [
       123,
       12,
       56
      ]
    ]
},
``

NB: this list has been specified as part of the [Jamoma project's Dataspace Lib](https://github.com/jamoma/JamomaCore/tree/master/Foundation/extensions/DataspaceLib), which has been used as specifications for [libossia unit management](https://github.com/OSSIA/libossia/tree/master/OSSIA/ossia/network/dataspace) - also see units.md

