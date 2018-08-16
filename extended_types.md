# EXTENDED_TYPEs

Some values' types sometimes need to be further described than what the OSC typetags allow for.
For example, if your address space has a method that accepts a filepath as a string, you could set the "EXTENDED_TYPE" to "filepath", and other software that inspects this could provide a file picker UI instead of just a blank text field.

Here is a proposed list of such extended types, which can be documented using the EXTENDED_TYPE field, sorted by their corresponding OSC type tags :

### s

- filepath: string describing a UNC or POSIX file path
- url: string representing an URL or URI

### T, F, int (with RANGE [0, 1])

- bool

### arbitrary list of various types:

- list 

### arbitrary arrays of strings:

- stringArray

### arbitrary arrays of floats:

- floatArray

### arbitrary arrays of ints:

- integerArray

of which, some like

### ff, fff, ffff or ii, iii, iiii, aka:

can describe multi-dimensional quantities, which we will discuss in the section below:

## Multi-dimensional quantities  

Similarly (and complementary) to **UNIT**s for scalars (aka single-dimension quantities), vectors (aka multi-dimensional quantities) can be classified into sets of mutually convertible values.


We propose to use the EXTENDED_TYPE field for documenting those.


Also, those quantities being multi-dimensional, their members' units can and must (according to the OSCQuery specs) be declared individually. In the list below, the elements between curly brackets indicate pairs of:
- each member's name (to be appended to the multi-dimensional EXTENDED_TYPE), e.g. ```position.cart2D``` 
- the **UNIT** category to which it belongs .



### Position: (2 or 3 dimensions)
- position.cart2D {x:distance, y:distance}
- position.polar {a:angle, d:distance} 
- position.cart3D {x:distance, y:distance, z:distance}
- position.spherical {a:angle, e:angle, d:distance} 
- position.cylindrical {d:distance, a:angle, z:distance}
- position.openGL {x:distance, y:distance, z:distance} (inverted Z and Y axis, compared to cart3D)


### orientation: (3 or 4 dimensions)
- orientation.quaternion {a:distance, b:distance, c:distance, d:distance} ( extension of the complex numbers for 3D orientation, in the form a+bi+cj+dk)
- orientation.euler {y:angle, p:angle, r:angle} (yaw, pitch, roll: describing the orientation of a rigid body with respect to a fixed coordinate system)
- orientation.axis {x: distance, y:distance, z:distance, w:angle} (An angle (w) relative to a 3-dimensional axis of vector XYZ)

### Color:
- color.rgba8 (8-bit RGBA color, corresponding to the OSC "r" type)
- color.rgba (4-float vector for RGBA)
- color.rgb
- color.bgr
- color.argb
- color.argb8
- color.hsv
- color.cmy8 (8-bit cyan, magenta, yellow color)
- color.xyz


# An example:
A Spherical position could be described this way in a OSCQuery JSON:

```
"position" : {
    "EXTENDED_TYPE" : [
         "position.spherical.a",
         "position.spherical.e",
         "position.spherical.d",
    ]
    "FULL_PATH" : "\/some\/position",
    "RANGE" : [
         {
           "MAX" : 0,
           "MIN" : 360
         },
         {
           "MAX" : 0,
           "MIN" : 360
         },
         {
           "MAX" : 0,
           "MIN" : 500
         }
    ],
    "TYPE" : "fff",
    "UNIT" : [
        "angle.degree",
        "angle.degree",
        "distance.m"
     ]
    "VALUE" : [
       123,
       12,
       31
    ]
},
```



