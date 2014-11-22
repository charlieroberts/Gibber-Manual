# Audio Part 1: Oscillators & Modulation

Gibber includes a number of raw oscillators to experiment with and use as LFOs.

- Sine
- Triangle
- Square
- PWM (anti-aliased)
- Saw (anti-aliased)
- Noise

The constructors for each of most of these oscillators accept frequency and amplitude arguments. So to create a Sine oscillator with a frequency of 220 Hz and an amplitude of .1:

```js
a = Sine( 220, .1 )
```

Alternatively we can pass a dictionary to the constructor and selectively list properties in any order we choose:

```js
a = Sine({ frequency:220, amp:.1 })
```

Note that as soon as we create the Sine oscillator, it is immediately connected to Gibber's master output. This is different from most audio programming environments. To disconnect an individual oscillator, you can either use the `disconnect()` method or, as a convenience, access the `_` underscore property.

```js
// create and connect
a = Sine()
// disconnect
a.disconnect()
// reconnect
a.connect()
// disconnect()
a._
```

##Modulation##
The PWM oscillator (pulsewidth modulation) has an additional pulsewidth property <sup>1</sup>. We can easily modulate the pulsewidth property with another oscillator in Gibber as long as we remember one very important fact: **we must disconnect the modulating oscillators from the master output.** This is important enough that I'll say it again:

**You must disconnect typical LFOs from the master output**.

Why? For pulsewidth modulation it might not be a big deal, as we'll only be moving through a range of 0 - 1. But frequency modulations might occupy a much wider range, and connecting an oscillator with an amplitude of 50 to the master output will not be pleasant.

This is a huge downside to Gibber's 'connect on instantiation' approach. I'm still not sure if it is the right way to do it (most other audio programming environments would say it's not), but that's the way it is for now.

So. With that said, let's modulate:

```js
a = PWM()
b = Sine( 4 )._ // disconnect!!!
a.pulsewidth = b
```

Note that this is actually creating a pulsewidth modulation between -1 to 1. If we really want a range of 0 - 1, we need to use the `Add()` unit generator, which simply adds two signals, to create an offset of .5.

```js
a = PWM({
  pulsewidth: Add( .5, Sine( 2, .5 )._ ) // disconnect!
})
```

###Modulating Modulators###

Of course, we can also modulate our modulation. In the example below, we create vibrato that gradually changes in depth over time.

```js
a = Sine()
a.frequency =
  Add(
    440,
    Sine( 4, Sine( .01, 100)._ )._ // disconnect! twice!
  )
```

To change operands to a math binop ugen, you can simply use the `[0]` and `[1]` accessors. For example:

```js
mod = Sine( .1, 100 )._
add = Add( 400, mod )
sine = Sine( add, .2 )

// change base frequency of 400 to 600
add[0] = 600
// change modulator frequency from .1 to 2
add[1].frequency = 2
```

Other math unit generators include:

- Mul (multiply two numbers / signals)
- Div (divide two numbers / signals)
- Abs (take the absolute value of one number / signal)
- Sub (subtract two numbers / signals)
- Pow (raise a number / signal to a power)
- Mod (module two numbers / signals)
- Sqrt (take the square root of a number / signal )

##A FM Synth##

Let's make an example FM Synth using JavaScript. This will not make much sense unless you know about FM synthesis; if you don't know FM you might want to [try this resource](http://www.soundonsound.com/sos/apr00/articles/synthsecrets.htm). Gibber actually comes with a FM Synth built-in, but this will serve as an example of how to create a basic synth using raw oscillators and modulation.

In our FM synth, we'll use two oscillators, a *carrier* and a *modulator*. Their frequencies will be combined together using an `Add` unit generator. When we call the `note()` method of our synth, we'll change the base carrier frequency, the modulator frequency and the modulator amplitude.

```js
MyFM = {
    index: 5,
    carrierToModulationRatio: 2,
    frequency:440,
    amp: .2,
    carrier: null,
    modulator: null,
    init: function() {
        this.carrier = Sine( 440, this.amp )
        this.modulator = Sine( 440, this.index * this.frequency )._

        // create fm
        this.note( this.frequency )
    },
    note: function( newFrequency ) {
        this.frequency = newFrequency
        var modFreq = this.frequency * this.carrierToModulationRatio

        this.modulator.frequency = modFreq
        this.modulator.amp = this.index * this.frequency

        this.carrier.frequency = Add( this.frequency, this.modulator )
    }
}

// initialize our synth
MyFM.init()

// play some different notes
MyFM.note( 1000 )
MyFM.note( 220 )

// change the carrier to modulation ratio
MyFM.carrierToModulationRatio = 10.315

// play another note to hear change to timbre
MyFM.note( 330 )
```


####Footnotes####

1. For more info on pulse width modulation, [here's a reference on the Sound On Sound website](http://www.soundonsound.com/sos/Mar03/articles/synthsecrets47.asp).

