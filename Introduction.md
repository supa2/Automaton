* [Framework](#framework)
* [Reactive Programming ](#reactive-programming)
* [Fluent Interface](#fluent-interface)
* [Callbacks & lambda expressions](#callbacks--lambda-expressions)
* [Joining the Internet Of Things (IoT) with the esp8266](#joining-the-internet-of-things-with-the-esp8266)
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

We now have two component (they're actually state machine objects, but we can forget about that for now). Next we initialize these components in the Arduino setup() method.

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

Now we have two initialized components but they're in no way connected to each other. They are however connected to the 'automaton' object. When we call the begin() method on an Automaton component it will attach itself to the global automaton scheduler. This means it will be kept running as long as the automaton.run() method is regularly executed. The sketch so far will compile and run, be nothing will happen when you press the button connected to pin 2.

If we want a button press event to start the led blinking we specify that by extending the setup method like this:

```c++
void setup() {
  led.begin( 4 );
  button.begin( 2 );
  button.onPress( led, led.EVT_BLINK );
}
```

It would be nicer to make the blinking toggle on and off, fortunately that is easy to arrange because it's built into the led component, just change the event to EVT_TOGGLE_BLINK.

```c++
  button.onPress( led, led.EVT_TOGGLE_BLINK );
```

A full sketch with a blinking led and a button that toggles it on and off looks like this.

```c++
#include <Automaton.h>

Atm_led led;
Atm_button button;

void setup() {
  led.begin( 4 );
  button.begin( 2 )
    .onPress( led, led.EVT_TOGGLE_BLINK );
}

void loop() {
  automaton.run();
}
```

### Reactive Programming ###

As you see, a change in the button component leads to an automatic change in the led component. Like in a spreadsheet a change in a component (cell) automatically propagates to other components (cells). We call this 'reactive' programming. 

[Reactive Programming on wikipedia](https://en.m.wikipedia.org/wiki/Reactive_programming)

A pleasant side effect of this way of programming is that it is extremely easy to expand our simple sketch to two or three leds and buttons. It scales much easier in terms of sketch complexity than the typical Arduino code.

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

We now have two buttons independently controlling two leds. I told you it was easy...

### Fluent interface ###

Automaton has a so-called 'fluent interface'. Instead of repeating ```button.begin();``` and ```button.onPress();``` you can *chain* them together like this: ```button.begin().onPress();``` We usually insert newlines to put each method on its own line.

[Fluent Interface on wikipedia](https://en.m.wikipedia.org/wiki/Fluent_interface)

```c++
  void setup() {
    // Start a led blinking at 200ms on and 200ms off
    led.begin( 4 );
    led.blink( 200, 200 ); 
    led.start();

    // Now do the same using the 'fluent' syntax
    led.begin( 4 )
      .blink( 200, 200 )
      .start();

    // Or all on one line if you prefer compactness
    led.begin( 4 ).blink( 200, 200 ).start();
  }
```

This makes for more readable code. With the proper indenting it is immediately obvious that these commands are grouped and together they configure the 'led' component. This combination of event driven programming and fluent interface has been made popular by modern frameworks like jQuery and Node.js.

It will work either way and you can mix the fluent syntax with the more traditional object.method() calls. Use whatever fits best. Most Automaton methods return a reference to their own object, which is what makes this possible.

### Callbacks & Lambda expressions ###

Anytime you use one of the on* methods, like button.onPress(), you have the choice between sending an event to another component object and calling a callback function. You could use a callback to modify the example above so that one button toggles two leds.

```c++
void callback( int idx, int v, int up ) {
  led.toggle();
  led2.toggle();
}

void setup() {

  led.begin( 4 );
  led2.begin( 5 );

  button.begin( 2 )
    .onPress( callback );

}
```

The onPress() method is what we call a *Push Connector* because it connects the button object to a callback function in such a way that a change (press) in the button object results in the callback being triggered. The press event is being pushed to the callback. Callbacks used in a *Push Connector* always have the three *idx*, *v* and *up* integer arguments. Their function may vary from component to component, check the documentation for details.

Another newcomer in the Arduino world is the use of Lambda Expressions.
The use of a separate callback function makes the code in the example above appear more fragmented. By using a so called lambda expression (only available in the Arduino IDE since version 1.6.6) we can keep the code that defines the button nicely together.

```c++
void setup() {

  led.begin( 4 );
  led2.begin( 5 );

  button.begin( 2 )
    .onPress( [] ( int idx, int v, int up ) {
      led.toggle();
      led2.toggle();
    });

}
```

Just for comparison have a look at these jQuery fragments.

```javascript

// Method chaining here
$("#p1").css("color", "red")
  .slideUp(2000)
  .slideDown(2000);

// Lambda expressions (called closures in javascript)
$.get( "test.php", function( data ) {
  $( "body" )
    .append( "Name: " + data.name ) // John
    .append( "Time: " + data.time ); //  2pm
}, "json" );

```

Looks familiar, doesn't it?

### Joining the Internet Of Things with the esp8266

Automaton provides a way to separate processes into running components and to make them work
together in a clear event-driven way. It comes with a collection of bundled components to 
control and monitor digital pins, control relays, leds and piezo speakers. To receive input
from serial ports, rotary encoders, buttons and timers. We won't go into all of these in this
introduction.

But there's an Automaton extension for the popular range of development boards for the esp8266 
wifi-enabled microcontrollers that's worth taking an introductory look at. The Automaton-Esp8266 
extension makes it easy to connect your reactive sketches to the web using the same event-driven 
Automaton coding technique.

Starting with the button+led sketch we created above, we could extend our code to allow turning the blinking 
led on and off from a web browser by calling two urls.

```c++
#include <Automaton.h>
#include <Atm_esp8266.h>

Atm_esp8266_wifi wifi;
Atm_esp8266_httpd_simple server( 80 );

Atm_led led;
Atm_button button;

void setup() {

  led.begin( D6 );

  button.begin( D5 )
    .onPress( led, led.EVT_TOGGLE_BLINK );
    
  wifi.begin( "MySSID", "MyPASSWORD" ) // Connect to the WIFI network
    .onChange( true, server, server.EVT_START )
    .start();
    
  server.begin()  // Hello IoT!
    .onRequest( "/on", led, led.EVT_BLINK )
    .onRequest( "/off", led, led.EVT_OFF );
    
}

void loop() {
  automaton.run();
}
```

The new code loads the [Automaton-Esp8266 extension](https://github.com/tinkerspy/Automaton-Esp8266) and adds two esp8266 specific components. 
The first one ('wifi') makes the wifi connection and starts the second ('server', the web server component) as soon as a connection is established. 

When a HTTP request for the `/on` url comes in the webserver sends an EVT_BLINK event to the led component.
When a request for the `/off` url comes in, the led is turned off. It could hardly be simpler. A led simultaneously controlled by a toggle button and a web browser in just a few lines of fully multitasking code.

### Mix & match ###

You can combine Automaton's event driven code with regular Arduino code as long as you make sure that the automaton.run() method is regularly called to update the running state machines.

[Documentation for the Atm_led class](/tinkerspy/Automaton/wiki/The-led-machine)  
[Documentation for the Atm_button class](/tinkerspy/Automaton/wiki/The-button-machine)