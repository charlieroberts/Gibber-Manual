# Audio Part 3: Effects & Bussing

Gibber comes with a variety of audio fx to employ, all of which take advantage of the [sequencing abstractions](./sequencingPart1.html) and [mapping abstractions](./mapping.html) described in earlier chapters. The included FX are:

- Reverb
- Delay
- Flanger
- Chorus
- Distortion
- Crush (bit depth and sample rate quantization)
- Schizo (buffer shuffling, reversing, pitch shifting)
- Vibrato
- Tremolo
- RingMod
- HPF (high pass filter)
- LPF (low pass filter)

All oscillator and synthesizer unit generators contain an array of fx that you can freely add to / remove from. In addition, Gibber allows you to create dedicated busses for hosting effects, so that multiple synthesis units can send their signal to be summed into the same effect; this is especially useful for relatively expensive effects like reverb. The `Group` command can be used to group a selection of unit generators into a single bus with a single fx chain.

##Adding and Removing Effects##

Every synthesis unit generator has an array named `fx` with both `add` and `remove` methods. `add` simply adds a fx argument to the end of an fx chain, while `remove` removes a argument from the chain, or, if the argument is a string that names an effect, searches the fx chain for the type of effect identified and removes it.

```js
a = Synth().note.seq( 0, 1/2 )
b = Crush({ bitDepth:4 })

a.fx.add( b, Reverb() )

// fx can use Gibber's sequencing abstractions
b.sampleRate.seq( [.1,.2,.3,.4], 1/2 )

// fx can also use Gibber's mapping abstractions
b.bitDepth = Mouse.X

a.fx.remove( b )

a.fx.remove( 'Reverb' )

a.fx.add( b, Delay() )

// shortcut to remove all fx
a.fx.remove()
```

Although the `fx` property is in fact an array, it's probably best to not think of it that way; you need to call the `add` or `remove` methods (which are added to the array) to actually affect the signal processing chain. For example, setting the `fx` array length to 0, which deletes all items in the array, will have no effect on the signal processing graph. Similarly, standard javascript array methods like `push` and `pop` will also not affect the audio graph when called on an fx array.

```js
a = Drums('x*ox*xo-')

a.fx.add( Distortion(10), Flanger() )

// no effect
a.fx.length = 0
```

##Using Busses##

The `Bus` object allows us to route signals from any unit generator (either a synthesis ugen or an fx ugen) to a dedicated output object. The `Bus` object only has two properties of any interest: `amp` and `pan`. For expensive fx, it often makes sense to create a single instance and assign it to a `Bus` object, and then route all ugens that want to use the effect to the dedicated `Bus`. In the example below, we create two busses, each with a `Schizo` effect added and panned hard left and hard right. We then route a `Drums` object to feed the busses using the `send` method.

```js
drums = Drums('x*o*x*o-')

busRight = Bus().fx.add( Schizo('paranoid') ).pan( 1 ) // right
busLeft = Bus().fx.add( Schizo('paranoid') ).pan( -1 ) // left

a.send( busLeft, .75 ) // send 75% to busLeft
a.send( busRight, .75 ) // send 75% to busRight
```

Note that if you call `send` and pass `0` for the amount to send, this will override any previous calls to send and remove the connection to the bus. For example, to sequence sending a synth to an FX bus and then removing the connection:

```js
a = Synth().note.seq( Rndi(0,12), 1/8 )

b = Bus().fx.add( Delay() )

var toggle = 0
c = Seq( function() {
  toggle = !toggle // switch between 0 and 1
  a.send( b, toggle )
}, 2 )
```

`Bus` objects can be disconnected from the `Master` output in Gibber the same way as any other Gibber unit generator:

```js
a = Bus() // connected
a._       // same as a.disconnect()
a.connect( Master ) // reconnected
```
##The Group Object##

The `Group` command creates a new `Bus` object, and then disconnects every synthesis unit generator passed as an argument and connects it to itself. In effect, it removes connections to the `Master` output and instead patches them into a new bus. This makes it easy, for example, to quickly route a number of ugens into an effect.

```js
a = Drums('x*ox*xo-')
b = Pluck().note.seq( Rndi(0,12), 1/8 )
c = FM('bass').note.seq( Rndi(0,12), [1/8,1/16].rnd(1/16,2) )

// add fx to drums and synth but note fm
d = Group( a, b ).fx.add( Distortion(100) )

// reconnect drums to their previous bus
d.free( a )
```
