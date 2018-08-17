# EXTENDED_TYPE

## URLs

- `filepath`: string describing a UNC or POSIX file path
- `url`: string representing an URL or URI

## Multi-dimensional types

Following are some accepted values for the EXTENDED_TYPE attribute, for multi-dimensional types, which all use the same notation:
- Characters inside brackets enumerate the various possible values- for example, `position.polar.[rp]` means that you can use `position.polar.r`, and `position.cart.y`.
-  They can be used jointly (for all dimensions) or separately. A few examples are demonstrating this at the end of the document.


### Position

- `position.cartesian.[xyz]`- A cartesian point, consisting of any combination of x, y, and z components.
  - `x`- A number value describing a distance.
  - `y`- A number value describing a distance.
  - `z`- A number value describing a distance.
- `position.polar.[rp]`- A point in 2D space described using polar coordinates
  - `r`- A number value describing the radial distance.
  - `p`- A number value describing the azimuth angle.
- `position.spherical.[rtp]`- A point in 3D space described using spherical coordinates in the form suggested by ISO 80000-2:2009/ISO 31-11.
  - `r`- A number value describing the radial distance.
  - `t`- A number value describing the inclination angle.
  - `p`- A number value describing the azimuth angle.
- `position.cylindrical.[rpz]`- A point in 3D space described using cylindrical coordinates in the form suggested by ISO 80000-2:2009/ISO 31-11.
  - `r`- A number value describing the radial distance.
  - `p`- A number value describing the azimuth angle.
  - `z`- A number value describing the height.

### Orientation

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

### Color

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

### Examples 

- A “canonical usage of cartesian position coordinates”:
~~~json
"position": {
	"FULL_PATH": "/position",
	"TYPE": "fff",
	"EXTENDED_TYPE": [
		"position.cart.x",
		"position.cart.y",
		"position.cart.z",
	],
}
~~~
- Not all of these attributes are required to be used- for example, `position.cart.[xyz]` potentially includes `position.cart.x`, `position.cart.y`, and `position.cart.z`, but you don't need to use all of these at the same time.  The following examples are all correct:
~~~json
"2d_position": {
	"FULL_PATH": "/2d_position",
	"TYPE": "ff",
	"EXTENDED_TYPE": [
		"position.cart.x",
		"position.cart.y",
	],
}
~~~
~~~json
"x": {
	"FULL_PATH": "/pos/x",
	"TYPE": "i",
	"EXTENDED_TYPE": [ "position.cart.x" ]
},
"funky_2D": {
	"FULL_PATH": "/pos/funky_2D",
	"TYPE": "ff",
	"EXTENDED_TYPE": [ "position.cart.z", "position.cart.y" ]
}
~~~
