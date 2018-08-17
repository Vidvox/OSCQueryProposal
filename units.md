# UNIT

Following are some proposed examples of accepted values for the UNIT attribute:

## Distance:
- `distance.m`
- `distance.km`
- `distance.dm`
- `distance.cm`
- `distance.mm`
- `distance.um`
- `distance.nm`
- `distance.pm`
- `distance.inches`
- `distance.feet`
- `distance.miles`
- `distance.pixels`

## Angle:
- `angle.degree`
- `angle.radian`

## Gain:
- `gain.linear` (a normalized [0. 1.] range mapping to (-inf 0dB] )
- `gain.midigain`  (a MIDI-adapted gain, percentage-like - recommended mapping (midigain/dB pairs): {0, inf}, {100, 0dB}, {127, +12dB} )
- `gain.db` (clipped to a minimum headroom value)
- `gain.db-raw` (not clipped)

## Time (and pitch):
- `time.second`
- `time.bark`
- `time.bpm`
- `time.cents`
- `time.hz`
- `time.mel`
- `time.midinote` (usual MIDI note convention, in the range [0-127], with 36 being 440Hz)
- `time.ms`
- `time.speed`
- `time.samples`

## Speed:
- `speed.m/s`
- `speed.mph`
- `speed.km/h`
- `speed.kn`
- `speed.ft/s`
- `speed.ft/h`
