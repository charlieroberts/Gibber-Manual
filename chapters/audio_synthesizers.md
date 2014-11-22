# Audio Part 2: Synthesizers

*Synthesizers* in Gibber refer to, at a minimum, raw oscillator(s) attached to an amplitude envelope. The five synthesizers in Gibber are:

- `Synth` ( oscillator + envelope )
- `Synth2` ( oscillator + envelope + filter )
- `Mono` ( three oscillators + envelope + filter )
- `FM` ( two-operator FM synthesis + envelope )
- `Pluck` ( Karplus-Strong algorithm )

All synthesizers in Gibber have a `note` method, which accepts frequency and amplitude arguments. Here is a basic sequence using a synth:

```js
a = Synth().note.seq( [0,1,2,3], 1/4 )
```

In the above example, the synth will play the first four notes of whatever Gibber's current global scale is, with each note lasting a 1/4 note.

##Envelopes##

All synths except `Pluck` have an adjustable Attack / Decay envelope attached to them that can be switched to Attack / Decay / Sustain / Release behavior.

```js
a = Synth({
  attack: ms(1),
  decay: 1/2
})
.note.seq( [0,1,2,3], 1/4 )
```

If the ADSR is used, by default you need to call the `note` method with amplitude `0` for the envelope to release.

```js
a  = Synth({
  useADSR: true
})

a.note( 440, .5 ) // 440 Hz, .5 amplitude
// wait a couple of seconds
a.note( 440, 0 ) // 0 amplitude triggers release
```

To use an ADSR envelope that doesn't require a `note` message with 0 amplitude to release, you can set the `requireReleaseTrigger` property to `false`.

```js
a = Synth({
  useADSR:true,
  requireReleaseTrigger:false,
  attack:ms(1), decay:ms(50), sustain:ms(500), release:ms(50)
})
.note.seq( [0,1,2,3], 1 )
```

###Pluck Envelope##
The envelope for the `Pluck` synthesizer is determined by the algorithm it uses to generate sound <sup>1</sup>; there is no control over the attack. However, you can adjust the decay by changing the `damping` property of the synth.

```js
a = Pluck()
  .note.seq( 0, 1/2 )
  .damping( [.25,.5,.75,1] )
```

##Polyphony##

By default, all synths are monophonic for efficiency reasons. You can change this using the `maxVoices` property of the synths. Once `maxVoices` is set to a value higher than 1, you can then use `chord` method to trigger multiple notes simultaneously.

```js
a = FM({ maxVoices:4 })

a.chord( [0,2,4,6] )

a.chord.seq( Rndi(0,6,3), 1/2 )
```

Gibber uses a very simple voice queue for voice allocation. Whenever a new note is triggered, the voice that was triggered furthest in the past is selected to play the new note.

##Pan##
All synths are stereo, and all synths have a `pan` property (with a range of {-1,1}) that can be used to position their output in the stereo spectrum.

```js
a = Mono()
  .note.seq( 0, 1/4 )
  .pan.seq( [-1, -.5, 0, .5, 1] )

// smoothly move across stereo spectrum by modulating pan
b = FM({
  attack:44,
  pan:Sine( .1 )._
})
.note.seq( 14,1/4 )
```

##Synth Descriptions##

###Synth###
The most basic synth, this consists of a raw oscillator connected to an amplitude envelope. You can change the `waveform` property to be any one of the following types:

- Sine
- Square
- Triangle
- PWM
- Saw
- Noise

```js
a = Synth().note.seq( 0, 1/2 )

var num = 0, waves = ['Saw','PWM','Sine','Triangle']

b = Seq( function() {
  a.waveform = waves[ num++ % waves.length ]
}, 1/2 )
```

`Synth` (as well as `Synth2`, `Mono` and `FM`) also has a `glide` property that causes notes to smoothly transition between pitches (also known as portamento). The `glide` property actually sets a filter coefficient on a low-pass filter; values that are close to `1` (like `.99999`) have longer transition times. Values below `.9` produce transitions that are to quick to be noticeable.

The `Synth` class has four presets: short, bleep, rhodes and calvin.

###Synth2###
`Synth2` has the same properties as `Synth`, but with the addition of a 24db 'Moog-style' ladder filter. The filter cutoff is tied to the same envelope that controls the amplitude, however, you can adjust the base `cutoff` value used along with the filter's `resonance`.

```js
a = Synth2({ maxVoices:4, amp:.5, resonance:4 })
  .chord.seq( [ [0,1,2,4] ], 1/2 )
  .cutoff.seq( [.1,.2,.3,.4], 1/2 )
```

`resonance` can be set to arbitrarily high values, however, in practice it is only safe to use values up to about `4.5` or so. `cutoff` is measured from {0,1}.

###Mono###
`Mono` is a three-oscillator monosynth based on architecture of a MiniMoog. The second and third oscillator frequency are tuned as offsets of the first; you can set both an octave offset and and a detune amount. The filter properties are the same as `Synth2`, and it also shares the same amplitude envelope and `glide` properties.

```js
a = Mono()
    .note.seq( 0, 1/4 )
    .detune2.seq( Rndf(0,.015) )
    .detune3.seq( Rndf(0,.015) )

// set oscillator octaves to match main oscillator
a.octave2 = 0
a.octave3 = 0
```

The `Mono` class has numerous presets: short, lead, winsome, bass, dark, dark2, easy, easyfx, noise.

###FM###

This is a simple, two-operator FM synthesis patch<sup>2</sup> feeding into an amplitude envelope. There is no filter attached. The important FM properties are:

- `cmRatio` : The ratio between the carrier and modulation frequencies. This ratio is maintained as different notes are triggered on the synth.
- `index` : The amplitude of the modulator equals the frequency of the carrier times whatever this value is. Larger values tend to produce timbres with more spectral complexity.

For example, a FM patch considered to be 'brassy' has a `cmRatio` of 1/1.0007 and an `index` property of 5.

Gibber comes with numerous presets for the FM synth: gong, drum, drum2, brass, bass, clarinet, glockenspiel, noise and stabs.

```js
a = FM( 'bass' ).note.seq( [0,7], 1/8 )

b = FM( 'stabs' )
    .chord.seq( Rndi(0,6,3), [1/4,1/8,1/2].rnd() )

c = Drums('x*o*x*o-')

d = FM( 'glockenspiel' )
  .note.seq( Rndi(0,12), [1/4,1/2,1,1/8].rnd() )
  .fx.add( Delay() )

d.index = c.Out

G.scale.root.seq( ['c4','ab3'], 2 )
```

###Pluck###
The `Pluck` synth has a whopping total of two extra properties: `blend` and `damping`. The `blend` property introduces noise into the signal at low values by randomly flipping the sign of individual samples. This can be used to easily create some sounds that are characteristic of hi-hats (the code below contains one of thecharlie's secret Gibber recipes):

```js
a = Pluck()
	.note.seq( Rndi(0,12), [1/4,1/8,1/2].rnd() )
	.fx.add( Vibrato(2) )

b = Pluck().note.seq( Rndi(200,600), 1/16 )
  .blend.seq( Rndf() )
  .fx.add( Schizo('paranoid') )
  .pan.seq( Rndf(-1,1) )
```

####Footnotes

1: http://en.wikipedia.org/wiki/Karplus–Strong_string_synthesis

2: http://www.soundonsound.com/sos/apr00/articles/synthsecrets.htm
