* [Framework](#framework)
* [Programming style](#programming-style)
* [Fluent Interface](#fluent-interface)
* [Mix & Match](#mix--match)

### Framework ###

Automaton is an event driven framework that makes writing multi tasking Arduino sketches easier than ever. It's called a framework instead of 'just' a library because it takes over the execution control of your sketch. Typically, you no longer write your sketch code inside the Arduino 'loop' function.

Instead you initialize the components of your sketch (leds, buttons, servo's etc.) in the setup method and you specify how they should interact with each other when certain events occur. We call this 'Event driven programming'.

Let's have a look at a simple sketch with a blinking led and a button to turn the blinking on or off.

First we need to include the Automaton.h file and declare two objects, a button and a led. This is done at the top of the sketch.

```c++
#include <Automaton.h>

Atm_led led;
Atm_button button;

```

We now have two component (they're actually state machine object, but you can forget about that for now). Next we initialize these components in the Arduino setup() method.

```c++
void setup() {
  led.begin( 4 );
  button.begin( 2 );
}
```

Now the led is connected to pin 4 and the button is connected to pin 2.

To complete the sketch we need a loop() function. The loop function for an Automaton sketch typically looks like this.

```c++
void loop() {
  automaton.run();
}
```

Now we have two initialized components but they're in no way connected to each other. They are however connected to the 'automaton' object. When we call the begin method on a component it will automatically attach itself to the global automaton object which means it will be kept running as long as the automaton.run() method is regularly executed. The sketch so far will compile and run, be nothing will happen when you press the button connected to pin 2.

Now we want a button press event to trigger the led's 'EVT_BLINK' event. We specify that be extending the setup method:
```c++
void setup() {
  led.begin( 4 );
  button.begin( 2 );
  button.onPress( led, led.EVT_BLINK );
}
```
It would be nicer to make the blinking toggle on and off, fortunately that is easy to arrange, just change the event to EVT_TOGGLE_BLINK.

```c++
void setup() {
  led.begin( 4 );
  button.begin( 2 );
  button.onPress( led, led.EVT_TOGGLE_BLINK );
}
```

### Programming style ###

As you see, a change in the button component leads to an automatic change in the led component. Like in a spreadsheet a change in a component (cell) automatically propagates to other components (cells). We call this 'reactive' programming. 

A pleasant side effect of this way of programming is that it is extremely easy to expand our simple sketch to two or three leds and buttons. It's much easier to scale than the typical Arduino code.

```c++
void setup() {
  led.begin( 4 );
  button.begin( 2 );
  button.onPress( led, led.EVT_TOGGLE_BLINK );
  led2.begin( 5 );
  button2.begin( 3 );
  button2.onPress( led2, led.EVT_TOGGLE_BLINK );
}
```
We now have two buttons independently controlling two leds.

### Fluent interface ###

Automaton has a so-called 'fluent interface'. Instead of repeating button.begin() and button.onPress() you can *chain* them together like this: button.begin().onPress() We insert newlines to make this more readable.

```c++
  void setup() {
    // Start a led blinking at 200ms on and 200ms off
    led.begin( 4 );
    led.blink( 200, 200 ); 
    led.start();

    // Do the same using the 'fluent' syntax
    led.begin( 4 )
      .blink( 200, 200 )
      .start();
  }
```

This makes for more readable code. With the proper indenting it is immediately obvious that these commands are grouped and together they configure the 'led' component. This combination of event driven programming and fluent interface has been made popular by modern frameworks like jQuery and Node.js.

### Mix & match ###

You can combine Automaton's event driven code with other code as long as you make sure that the automaton.run() method is regularly called to update the running state machines.

[Documentation for the Atm_led class](/tinkerspy/Automaton/wiki/The-led-machine)  
[Documentation for the Atm_button class](/tinkerspy/Automaton/wiki/The-button-machine)
