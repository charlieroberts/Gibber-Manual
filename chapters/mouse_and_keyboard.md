#Interaction: Mouse & Keyboard#

##Mouse##
The mouse and the keyboard provide easy ways to interact with audiovisual content. Gibber applies [its mapping abstractions](mappings.html) to the `Mouse` object making it simple to map the current mouse position to audio, geometry, and shader properties. For example:

```js
// audio, press mouse button to hear drums
a = Drums('xoxo')
a.pitch = Mouse.X
a.amp = Mouse.Button

// geometry
b = Cube()
b.scale = Mouse.Y

// shader
c = Dots()
c.scale = Mouse.X
```

The first time you make a continuous mapping using the `Mouse` object, Gibber automatically turns mouse tracking on. This can also be manually accomplished using the `Mouse.on()` command, while `Mouse.off()` will stop Gibber from tracking the mouse position.

The mouse position is a function of the browser window's width and height, and is always normalized to a range of `{0,1}`. When drawing directly to the `Canvas` object in [2D graphics mode](2d.html), you'll need to explictly look up the mouse position, typically on the `draw` method of the `Canvas` object. The example below turns mouse tracking on and then uses the mouse position to draw a series of randomly colored squares. Note you have to multiply the mouse position by the width and height of the canvas to position the squares correctly.

```js
Mouse.on()

a = Canvas()

a.draw = function() {
  var squareSize = 200
      xCoord = (Mouse.x * a.width) - squareSize / 2,
      yCoord = (Mouse.y * a.height) - squareSize / 2

  a.fade( .1 )
  a.square( xCoord, yCoord, squareSize ).fill( a.randomColor() )
}
```

The `Mouse` object also has `prevX` and `prevY` properties that report the mouse position in the previous frame of video. You can use this to calculate velocities or [draw mouse trails](http://gibber.mat.ucsb.edu/?p=gibber/Mouse Trails*2d*).

Other mouse properties include:

- shiftX / shiftY - the current x/y positions of the mouse if the shift key was down
- prevShiftX / prevShiftY - the previous x/y positions of the mouse when the shift key was down
- button -`1` when the mouse button is down, `0` when it is not.

Finally, you can add a `onvaluechange` method to the mouse object that will be called whenever the mouse moves or the button is pressed / released.

```js
a = Synth({ useADSR:true })

var prevValue = 0
Mouse.onvaluechange = function() {
  if( this.button !== prevValue ) {
    if( this.button ) {
      a.note( 440, .5 )
    }else{
      a.note( 440, 0 )
    }
  }
  prevValue = 1
}
```

##Keyboard shortcuts##

Gibber wraps [the excellent Mousetrap library](http://craig.is/killing/mice) to handle key events. This makes it easy to bind functions to user-defined keystroke combinations.

```js
a = Synth()

Keys.bind( 'ctrl+shift+l', function() {
  a.note(440)
})

// bind to multiple keys in a row
Keys.bind( 'z z z', function() {
  var editor = Layout.getFocusedColumn().editor

  editor.setValue( editor.getValue() + 'row row row your boat' )
})
```

For more information on what is possible, see the [Mousetrap documentation](http://craig.is/killing/mice).
