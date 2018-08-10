
# Here is a list of recommended values for the UNIT field:

It is composed of several categories, containing mututally convertible sets of **UNIT**s.

When the meaning of the **UNIT** is not straightforward, it is explained in parentheses, with an optional recommended implementation guideline.


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



NB: this list has been specified as part of the [Jamoma project's Dataspace Lib](https://github.com/jamoma/JamomaCore/tree/master/Foundation/extensions/DataspaceLib), which has been used as specifications for [libossia unit management](https://github.com/OSSIA/libossia/tree/master/OSSIA/ossia/network/dataspace) - also see vectors.md


