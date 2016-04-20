Atm_button is a state machine for implementing buttons. Connected to a digital input pin (in *PULLUP* mode) it will handle button presses, releases and holds. You'll need a single Atm_button instance for each button you want to service.

* [begin()](#atm_button--begin-int-attached_pin-)
* [onPress()](#atm_button--onpress-machine--machine-int-msg-)
* [debounce()](#atm_button--debounce-int-delay-)
* [longPress()](#atm_button--longpress-int-max-int-delay-)
* [repeat()](#atm_button--repeat-int-delay-int-speed-)
* [autoPress()](#atm_button--autopress-int-delay-int-press-)
* [onSwitch()](#machine--onswitch-swcb_sym_t-callback-)

## Synopsis ##

```c++
#include <Automaton.h>
#include <Atm_button.h>

Atm_button btn; 
Factory factory; 

void btn_change( int press ) 
{
  if ( press ) {
    // Do something when button is pressed
  }
}

void setup() 
{
  btn.begin( 11 );
  btn.onPress( btn_change );
  factory.add( btn );
}

void loop() 
{
  factory.cycle();
}
```

### Atm_button & begin( int attached_pin ) ###
Alternative: Atm_button & begin( int attached_pin, presscb_t press_callback ) 

Initializes an Atm_button object and attaches it to an I/O pin and a callback routine. For every keypress event the callback routine will be called. The I/O pin will be placed in *INPUT_PULLUP* mode so the hardware button connected should connect the pin to ground when pressed.

```c++
Atm_button btn;

void btn_change( int press ) 
{
  if ( press ) {
    led1.trigger( led1.state() ? led1.EVT_OFF : led1.EVT_BLINK );  // Toggle
  }
}

void setup() 
{
    btn.begin( 11, btn_change );
}
```

The press argument contains 0 if the event is a button release and 1 if the event is a button press. When in *longpress mode* the press argument may contain other values.

### Atm_button & onPress( Machine * machine, int msg ) ###
Alternative: Atm_button & onPress( Machine * machine, int msg_press, int msg_release )  
Alternative: Atm_button & onPress( presscb_t press_callback )  
Alternative: Atm_button & onPress( presscb_id_t press_callback, int idx )  

Registers a callback or a message destination to be called when a button is pressed.

Specifying a callback:

```c++
void btn_change( int press ) 
{
  if ( press ) {
    led1.trigger( led1.state() ? led1.EVT_OFF : led1.EVT_BLINK );  // Toggle
  }
}

void setup() 
{
  btn.begin( 11 );
  btn.onPress( btn_change );
}
```

Alternatively pass an idx parameter to the callback to reuse a single callback for multiple Atm_button objects:

```c++
void btn_change( int press, int idx ) 
{
  if ( press ) {
    if ( idx == 1 ) led1.trigger( led1.state() ? led1.EVT_OFF : led1.EVT_BLINK );  // Toggle
    if ( idx == 2 ) led1.trigger( led1.state() ? led1.EVT_OFF : led1.EVT_BLINK );  // Toggle
  }
}

void setup() 
{
  btn.begin( 11 ).onPress( btn_change, 1 );
  btn.begin( 12 ).onPress( btn_change, 2 );
}
```



Messaging another machine on a button press (pressing the button will start a led blinking):

```c++
btn.onPress( &led1, led1.MSG_BLINK );
```

Messaging another machine on a button press & release (a led will be lit as long as the button is pressed):

```c++
btn.onPress( &led1, led1.MSG_ON, led1.MSG_OFF );
```

The nice thing about using onPress() with a message argument is that a callback routine is not required. If you want to use the longpress mode you must use a callback to handle all the different button press events.

### Atm_button & debounce( int delay ) ###

Adds a debounce delay of *delay* milliseconds. Default debounce delay is 5 milliseconds.


```c++
void setup() 
{
    btn.begin( 11, btn_change );
    btn.debounce( 10 );
}
```

### Atm_button & longPress( int max, int delay ) ###

Switches the Atm_button machine to *longpress* mode. In *longpress* mode the events are not triggered by button presses but by button releases. This leads tio a slightly slower response, but the advantage is that we can let one button serve several functions.

The *delay* parameter controls the time the user should keep the button pressed (in millis) for the next event to register. The *max* parameter controls how many events are handled by the button. 

```c++
void setup() 
{
    btn.begin( 11, btn_change );
    btn.longPress( 2, 200 );
}
```
For the example above the machine will listen for two events. One if the button is pressed and released withing 200 milliseconds (fires -1, 1, 0 events). A second if the button is pressed, held for at least 200 ms and then released (fires -1, -2, 2, 0 events). If *max* is changed to 3, the machine will listen for another 200 ms hold.

In *longpress* mode the values received by the callback routine are no longer just 0 (release) and 1 (press).

Event code received | Meaning
------------ | -------------
0| The machine registered a release event
-1..-n | The machine passed through a threshold
1..n | The machine registered a press event

Normally your would only handle the positive 1..n events to respond to. The negative -1..n events may be useful to provide feedbackto the user by triggering a short beep from a buzzer. (buzzers can be triggered with Atm_led).

### Atm_button & repeat( int delay, int speed ) ###

Makes the button auto-repeat just like the keys on your keyboard. This is great for navigation keys controlling menus and such. Obviously this is not available in *longpress* mode. Calling *repeat() without parameters will select a 500 millisecond delay and a 50 millisecond repeat interval, which feels just like my keyboard.

```c++
void setup() 
{
    btn.begin( 11, btn_change );
    btn.repeat();
}
```

The *delay* parameter controls the delay before the first button repeat (default 500 ms). The *repeat* argument controls the time between repeats (default 50).

### Atm_button & autoPress( int delay, int press ) ###

For testing purposes the machine may be configured to automatically generate *press* events with the *autoPress()* method. The *delay* parameter controls the delay between presses. The *press* parameter is passed straight to the callback routine as a parameter.

```c++
void setup() 
{
    btn.begin( 11, btn_change );
    btn.autoPress( 1000, 1 );
}
```
### Machine & onSwitch( swcb_sym_t callback ) ###

To monitor the behavior of this machine you may connect a monitoring function with the Machine::onSwitch() method. 

```c++
btn.onSwitch( atm_serial_debug::onSwitch );
```



