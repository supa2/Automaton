Atm_button is a state machine for implementing buttons. Connected to a digital input pin (in *PULLUP* mode) it will handle button presses, releases and holds. You'll need a single Atm_button instance for each button you want to service.

![Button](images/button-medium.jpg)

<!-- md-tocify-begin -->
* [begin()](#atm_button--begin-int-attached_pin-)  
* [onPress()](#atm_button--onpress-connector-connector-argument-)  
* [onRelease()](#atm_button--onrelease-connector-connector-argument-)  
* [debounce()](#atm_button--debounce-int-delay-)  
* [longPress()](#atm_button--longpress-int-max-int-delay-)  
* [repeat()](#atm_button--repeat-int-delay--500-int-speed--50-)  
* [autoPress()](#atm_button--autopress-int-delay-int-press--1-)  
* [trace()](#atm_button--trace-stream--stream-)  

<!-- md-tocify-end -->

## Synopsis ##

```c++
#include <Automaton.h>

Atm_button button; 

void button_change( int idx, int v, int up ) {
  if ( v ) {
    // Do something when the button is pressed
  }
}

void setup() {
  button.begin( 2 )
    .onPress( button_change );
}

void loop() {
  automaton.run();
}
```

### Atm_button & begin( int attached_pin ) ###

Initializes an Atm_button object and attaches it to an I/O pin. The I/O pin will be placed in *INPUT_PULLUP* mode so the hardware button connected should connect the pin to ground when pressed.

```c++
#include <Automaton.h>

Atm_button button;
Atm_led led;

void setup() {
  led.begin( 4 );
  button.begin( 2 )
    .onPress( led, led.EVT_TOGGLE );
}

void loop() {
  automaton.run();
}
```

### Atm_button & onPress( {connector}, {connector-argument} ) ###

Registers a callback or a machine event to be triggered whenever a button is pressed.

Specifying a callback:

```c++
void button_change( int idx, int v, int up ) {
  if ( v ) {
    led.trigger( led.EVT_TOGGLE );  // Toggle
  }
}

void setup() {
  led.begin( 4 );
    button.begin( 2 )
      .onPress( button_change );
}
```
The v argument 1 if the event is a button press. When in *longpress mode* the v argument may contain other values. 

Alternatively pass an idx parameter to the callback to reuse a single callback for multiple Atm_button objects. Default value for idx is 0.

```c++
#include <Automaton.h>

Atm_button button1, button2;
Atm_led led1, led2;

void button_change( int idx, int v, int up ) {
  if ( idx == 1 ) led1.trigger( led1.EVT_TOGGLE_BLINK );  
  if ( idx == 2 ) led2.trigger( led2.EVT_TOGGLE_BLINK );  
}

void setup() {
  button1.begin( 2 )
    .onPress( button_change, 1 );
  button2.begin( 3 )
    .onPress( button_change, 2 );
}

void loop() {
  automaton.run();
}
```

Triggering another machine on a button press (pressing the button will start a led blinking). 

```c++
  led1.begin( 4 );
  button.begin( 2 )
    .onPress( led1, led1.EVT_BLINK );
```


The nice thing about using onPress() with an event argument is that a callback routine is not required. If you want to use the longpress mode you'll have to use a callback to handle all the different button press events.

### Atm_button & onRelease( {connector}, {connector-argument} ) ###

Registers a callback or a machine event to be triggered whenever the button is released.

### Atm_button & debounce( int delay ) ###

Adds a debounce delay of *delay* milliseconds. Default debounce delay is 5 milliseconds.


```c++
void setup() {
  btn.begin( 2 )
    .debounce( 10 );
}
```

### Atm_button & longPress( int max, int delay ) ###

Switches the Atm_button machine to *longpress* mode. In *longpress* mode the events are not triggered by button presses but by button releases. This leads to a slightly slower response, but the advantage is that we can let one button serve several functions.

The *delay* parameter controls the time the user should keep the button pressed (in millis) for the next event to register. The *max* parameter controls how many events are handled by the button. 

```c++



void setup() {
  Serial.begin( 9600 );
  btn.begin( 2 )
    .onPress( [] ( int idx, int v, int up ) {
      switch ( v ) {
        case 1:
          Serial.println( "Press 1" );
          return;
        case 2:
          Serial.println( "Press 2" );
          return;
      }
    })
    .longPress( 2, 200 );
}
```
For the example above the machine will listen for two events. One if the button is pressed and released within 200 milliseconds (fires -1, 1 events). A second if the button is pressed, held for at least 200 ms and then released (fires -1, -2, 2 events). If *max* is changed to 3, the machine will listen for another 200 ms hold.

In *longpress* mode the values received by the callback routine (in the v parameter) are:

Event code received | Meaning
------------ | -------------
-1..-n | The machine passed through a threshold
1..n | The machine registered a press event

Normally you would only handle the positive 1..n events to respond to. The negative -1..n events may be useful to provide feedback to the user by triggering a short beep from a buzzer. (buzzers can be triggered with Atm_led).

### Atm_button & repeat( int delay = 500, int speed = 50 ) ###

Makes the button auto-repeat just like the keys on your keyboard. This is great for navigation keys controlling menus and such. Obviously this is not available in *longpress* mode. Calling *repeat()* without parameters will select a 500 millisecond delay and a 50 millisecond repeat interval, which feels just like my keyboard.

```c++
void setup() 
{
  btn.begin( 2 )
    .onPress( led1, Atm_led::EVT_TOGGLE )
    .repeat();
}
```

The *delay* parameter controls the delay before the first button repeat (default 500 ms). The *repeat* argument controls the time between repeats (default 50).

### Atm_button & autoPress( int delay, int press = 1 ) ###

For testing purposes the machine may be configured to automatically generate *press* events with the *autoPress()* method. The *delay* parameter controls the delay between presses. The *press* parameter is passed straight to the callback routine the v parameter.

```c++
void setup() 
{
  btn.begin( 2 )
    .onPress( btn_callback )
    .autoPress( 1000, 1 );
}
```
### Atm_button & trace( Stream & stream ) ###

To monitor the behavior of this machine you may log state change events to a Stream object like Serial.

```c++
Serial.begin( 9600 );
btn.trace( Serial );
```

