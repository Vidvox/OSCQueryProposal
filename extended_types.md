# EXTENDED_TYPE

Following are some accepted string values for the EXTENDED_TYPE attribute.  There are no requirements or limitations on which of these values can be used with each other- an OSC method can be described using any number or combination of EXTENDED_TYPE strings.  Examples are provided at the end for clarity.

Characters inside brackets enumerate the various possible values- for example, `position.cartesian.[xyz]` means that there are three possible EXTENDED_TYPE strings: `position.cartesian.x`, `position.cartesian.y`, and `position.cartesian.z`.

#### Position

- `position.cartesian.[xyz]`- A cartesian point, consisting of any combination of x, y, and z components.
  - `x`- A number value describing a distance.
  - `y`- A number value describing a distance.
  - `z`- A number value describing a distance.
- `position.polar.[rp]`- A point in 2D space described using polar coordinates.
  - `r`- A number value describing the radial distance.
  - `p`- A number value describing the azimuth angle.
- `position.spherical.[rtp]`- A point in 3D space described using spherical coordinates.
  - `r`- A number value describing the radial distance.
  - `t`- A number value describing the inclination angle.
  - `p`- A number value describing the azimuth angle.
- `position.cylindrical.[rpz]`- A point in 3D space described using cylindrical coordinates.
  - `r`- A number value describing the radial distance.
  - `p`- A number value describing the azimuth angle.
  - `z`- A number value describing the height.

#### Orientation

- `orientation.quaternion.[abcd]`- A number system that extends complex numbers used for describing 3D orientation, in the form "*a* + *b*i + *c*j + *d*k" where *a*, *b*, *c*, and *d* are real numbers, and i, j, and k are quaternion units.
  - `a`- A number value describing a distance.
  - `b`- A number value describing a distance.
  - `c`- A number value describing a distance.
  - `d`- A number value describing a distance.
- `orientation.euler.[ypr]`- The orientation of an object described using euler angles.
  - `y`- A number value describing the yaw angle.
  - `p`- A number value describing the pitch angle.
  - `r`- A number value describing the roll angle.
- `orientation.axis.[xyzw]`- The orientation of an object described using an angle (w) relative to the 3-dimensional axis of vector xyz.
  - `x`- A number value describing the x component of the axis vector.
  - `y`- A number value describing the y component of the axis vector.
  - `z`- A number value describing the z component of the axis vector.
  - `w`- A number value describing the angle.

#### Color

- `color.rgb.[rgba]`- A color value described in the RGB space, potentially including an alpha channel.
  - `r`- A number describing the red channel value.
  - `g`- A number describing the green channel value.
  - `b`- A number describing the blue channel value.
  - `a`- A number describing the alpha channel value.
- `color.hsv.[hsva]`- A color value described in the HSV color space, potentially including an alpha channel.
  - `h`- A number describing the hue value.
  - `s`- A number describing the saturation value.
  - `v`- A number describing the "value" value (not a typo).
  - `a`- A number describing the alpha value.
- `color.cmyk.[cmyka]`- A color value described in the CMYK color space, potentially including an alpha channel.
  - `c`- A number describing the cyan value.
  - `m`- A number describing the magenta value.
  - `y`- A number describing the yellow value.
  - `k`- A number describing the black value.
  - `a`- A number describing the alpha value.
- `color.ciexyz.[xyza]`- A color value described in the CIE 1931 color space, potentially including an alpha channel.
  - `x`- A number value describing the x tristimulus value.
  - `y`- A number value describing the y tristimulus value.
  - `z`- A number value describing the z tristimulus value.
  - `a`- A number value describing the alpha value.

#### URLs

- `filepath`: string describing a UNC or POSIX file path
- `url`: string representing an URL or URI

# Examples

- An example of a single OSC method that accepts three floating-point values corresponding to the x, y, and z components of a cartesian position.
~~~json
"position": {
	"FULL_PATH": "/position",
	"TYPE": "fff",
	"EXTENDED_TYPE": [
		"position.cartesian.x",
		"position.cartesian.y",
		"position.cartesian.z",
	],
}
~~~
- Another example of a single OSC method that accepts a number of floating-point values- this time to describe a color in the CMYK color space.
~~~json
"position": {
	"FULL_PATH": "/color/subtractive",
	"TYPE": "fffff",
	"EXTENDED_TYPE": [
		"color.cmyk.c",
		"color.cmyk.m",
		"color.cmyk.y",
		"color.cmyk.k",
		"color.cmyk.a"
	],
}
 ~~~



- Not all of these attributes are required to be used- for example, `position.cartesian.[xyz]` potentially includes `position.cartesian.x`, `position.cartesian.y`, and `position.cartesian.z`, but you don't need to use all of these at the same time.  The following are both correct:
~~~json
"2d_position": {
	"FULL_PATH": "/2d_position",
	"TYPE": "ff",
	"EXTENDED_TYPE": [
		"position.cartesian.x",
		"position.cartesian.y",
	],
}
~~~
~~~json
"x": {
	"FULL_PATH": "/pos/x",
	"TYPE": "i",
	"EXTENDED_TYPE": [ "position.cartesian.x" ]
},
"funky_2D": {
	"FULL_PATH": "/pos/funky_2D",
	"TYPE": "fff",
	"EXTENDED_TYPE": [ "position.cartesian.z", "position.cartesian.y", "color.hsv.h" ]
}
~~~

- An example of a spherical position described using both EXTENDED_TYPE and UNIT.
~~~json
"position" : {
	"EXTENDED_TYPE" : [
		"position.spherical.r",
		"position.spherical.t",
		"position.spherical.p",
	],
	"FULL_PATH" : "/some/position",
	"RANGE" : [
		{	"MIN" : 0, "MAX" : 500	},
		{	"MIN" : 0, "MAX" : 360	},
		{	"MIN" : 0, "MAX" : 360	}
	],
	"TYPE" : "fff",
	"UNIT" : [
		"distance.m",
		"angle.degree",
		"angle.degree"
	],
	"VALUE" : [
		123,
		12,
		31
	]
},
~~~