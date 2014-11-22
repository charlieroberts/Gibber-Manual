#Audio, Visual and Interactive Mappings#

Gibber contains a special abstraction for creating mappings between properties of audio, visual and interactive objects <sup>1</sup> <sup>2</sup>. This abstraction came from thinking about the various modalities that Gibber employs and how they could quickly be combined.

The basic idea is simple: you can create a continuous mapping between any two properties of any two Gibber objects by simply captilizing the property name on the right side of the assignment operator.

```javascript
mysine = Sine(); mycube = Cube();

mycube.scale = mysine.Frequency

mysine.frequency.seq( [220,440,880,1760], 1/4 )
```

In the above example, since the `Frequency` property of our sine oscillator is capitalized during assignment (in the second line of code), a continuous mapping is made between the two elements. This means that as the frequency of our oscillator changes over time, the scale (size) of our cube will also change to reflect the new values.

Behind the scenes, the following actions take place when a this type of continuous mapping is made:

- Using metadata included about the properties of each object, the range of values on the righthand property (you could think of this as the output) is mapped to the range of values expected in the lefthand property (or input).

- Different forms of sample rate conversion handles the different timescales that the various modalities operate at. For example, if an audio property is mapped to a visual property, an envelope follower is placed on the value of the audio property so that a continuous average is used by the visual object. Conversely, a lowpass filter is placed on the values of visual properties when they are assigned to control audio properties.

- Linear -> logarithmic conversions are made as needed to be perceptually 'correct'. For exapmle, audio frequency is perceived
logarithmically, while rotation is perceived linearly. Hence, a mapping between freqeuency and rotation needs to accommodate  differing perceptual output curves.

The general idea is that Gibberers shouldn't have to concern themselves with these details, and only need to remember to capitalize that righthand property value to have these actions occur.

Gibber's capitalized notation for creating continuous mappings doesn't have to be used between objects of different modalities. You can also easily assign, for example, the pitch of a drum loop to be controlled by the drum loop's output envelope.

```js
mydrums = Drums('x*ox*xo-')
mydrums.pitch = mydrums.Out
```

Note that in the above example, and for audio objects in general, the `Out` property is added to objects to enable easy mappings that track the audio output envelope. However, the lowercase version, `out`, is not an real property value that can be read or changed. `Out` is unique in this fashion.

##Customizing Mappings##

Although Gibber uses metadata to try and ensure reasonable ranges of values for continuous mappings, these defaults are freely customizable using the `min` and `max` properties of the mapping objects. After a mapping object is made, it is stored in the capitalized property value for editing.

```js
a = Sine()
b = Drums('xoxo')
a.frequency = b.Out

a.Frequency.min = 200
a.Frequency.max = 300
```

Mappings can also easily be inverted:

```js
a.Frequency.invert()
```

Using [Gibber's sequencing techniques](sequencingPart1.html), we can easily schedule changes to the `min` and `max` properties as well as make calls to the invert method.

```js
a.Frequency.min.seq( [100,200,300], ms(500) )
a.Frequency.max.seq( [300,400,500], ms(500) )
a.Frequency.invert.seq( null, ms(1000) )
```

In the above example, the min and max values are changed every half a second while the mapping is inverted every second.

###Adjusting window size for envelope trackers###

When continuous mappings are made using audio properties, an envelope follower is placed on the audio property. The size of the envelope used can greatly impact the result of the mapping. For example, small envelopes can cause jerky responses in graphical properties, while longer envelopes can smooth out responses over time.

To adjust the size of an envelope, simply change the `env` property of a mapping object.

```js
a = Cube()
b = Drums('x*o*x*o-')

a.scale = b.Out
a.Scale.env = ms(100)
// wait a few seconds, and then try a larger envelope
a.Scale.env = ms(350)
```

##Mapping mapping properties (what?)##

Gibber allows the `min` and `max` properties of mapping objects to have their own continous mappings assigned. Below, we create a `Cube` and a `Drums` object. The output of the drums is continuously mapped to control its pitch. As the cube rotates along its y-axis, its rotation will control the upper bound of the pitch mapping.

```js
drums = Drums('x*ox*xo-')
drums.pitch = drums.Out

cube = Cube()
// continuously rotate on the y-axis
cube.mod('rotation.y', .01)

drums.Pitch.max = cube.rotation.Y
```

We can also imagine a situation where the rotation of the cube depends on the output of the drums, and the `max` value for the rotation is dependent on the drums pitch. These types of feedback networks fairly easy to create using the mapping abstraction described here.

##Some fun with the Mouse##

Although there will be another chapter on using interface elements with this mapping abstraction, for now we'll briefly point out how easy it is to continuously map the position of the mouse cursor in the editor window to control audiovisual objects.

```js
a = Knot()
b = Dots() // halftone shader
b.scale = Mouse.X // size of dots to x position
b.angle = Mouse.Y // angle of dot placement to y
```
##Summary##

- Continuous mappings can easily be created between Gibber's built-in audiovisual objects by capitializing property names during assignment
- Mapping ranges can be adjusted and inverted
- Changes and inversions of mappings can be sequenced.
- Mapping properties can continuously track other Gibber objects ("mappings all the way down")

####Footnotes####

1: It's actually an abstraction of an abstraction... the inspiration was research by Georg Essl in his [Urmus project](http://web.eecs.umich.edu/~gessl/georg_papers/ICMC10-UrSound.pdf).

2: Many of the concepts in this chapter are discussed in a [NIME  conference paper I authored ](http://charlie-roberts.com/pubs/Rapid_publication_nime_2014.pdf).

