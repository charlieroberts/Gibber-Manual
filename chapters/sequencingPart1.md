#Temporal Sequencing#

##Everything is a Sequence##
In musical programming, sequencing refers to programming sequences of musical notes and modulations. Regardless of whether you are composing with sound, light, or both, sequencing is often a critical component of time-based compositions. Gibber has a simple philosophy for sequencing:
'Everything is a sequence'<sup>1</sup>. In practice, this means that any property or method of any Gibber object can easily be sequenced. Let's look at an example of sequencing the frequency of a ``Sine`` wave. To start, we'll create a Sine oscillator with a frequency of 220 Hz. After changing the ``frequency`` property of the oscillator, you will hear a corresponding change in pitch, as demonstrated in the following two lines of code:

```js
a = Sine({ frequency:220 })
a.frequency = 880
```

If we want to change the frequency property at pre-defined intervals, we can use the ``seq`` method of the ``frequency`` property on our ``Sine`` object. In the following example, the oscillator will switch between two frequencies every 250 ms:

```js
a = Sine()
a.frequency.seq( [440, 880], ms(250) )
```

In the above code, we pass the ``seq`` method an array of *values* to sequence, and a single *duration*. We can also pass an array of durations to loop through. Let's consider changing the rotation of a 3D ``Knot`` geometry:

```js
a = Knot()
a.rotation.seq( [0, Math.PI / 2, Math.PI, Math.PI * 1.5], [ ms(100), ms(500), ms(1000) ] )
```

Note that in the above example, there are more values than durations. Upon reaching the end of either the values or durations array, the sequencer simply loops back to the beginning of that particular array.

Again, it's important to note that every property and method has the ``seq`` method available to it. This means that, in effect, each property / method is also its own object. *Property objects* in Gibber possess both the abstraction for sequencing that was just described, as well as a novel abstraction for creating multimodal, continuous mappings between properties that will be discussed in the next chapter. *Method objects* only possess the abstraction for sequencing. As an example of sequencing a method, consider the ``note`` method of the ``Synth`` object. It triggers a musical note at a provided frequency, with a corresponding amplitude envelope.

```js
a = Synth()
a.note( 440 )
a.note( 880 )
```

The ``note`` method can be sequenced in the same way we sequenced properties earlier.

```js
a = Synth()
a.note.seq( [220,440,880,1760], [1/4,1/8,1/16,1/2] )
```

##Chaining Sequences##
You can easily chain multiple sequences together; any call to the ``seq`` method of a property/method returns the object containing the property. For example, ``Sine().frequency.seq( [440,880], 1/4)`` would return the constructed ``Sine`` oscillator. I typically indent chained sequence calls for code readability:

```js
a = Drums('x*ox*xo-')
  .pitch.seq( [.5,1,2,4], 1/8 )
  .pan.seq( [-1,0,1], 1/8 )
  .shuffle.seq( null, 1 )
```

In the above example, four sequences are created. The first, a sequence of the ``note`` method, is actually created by default using the string passed to the ``Drums`` constructor. The second, sequencing ``pitch``, changes the speed that the drum samples are played at every 1/8th note. In the third, we change the ``pan`` of the drums every 1/8th note while in the final sequence, we call the ``shuffle`` method every measure, which randomizes the order of the original ``note`` sequence, creating changes in pattern.

A 3D geometry could be similarly sequenced:

```js
a = Cube()
  .rotation.seq( [.5,1,2,4], 1/8 )
  .position.x.seq( [-50,0,50], 1/8 )
  .scale.seq( [.5,1,2,4], 1/8 )
```

In the above example, we sequence the ``rotation`` of the cube along all three axes, the ``position`` of the cube along the x axis and the ``scale`` of the cube along all three axes.

##Summary##

Every property / method of Gibber objects has a corresponding ``seq`` method that can be used to easily sequence it.

####Footnotes####

1: Hat tip to grrrwaaa for this phrase

