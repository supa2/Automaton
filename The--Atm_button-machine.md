Atm_button is a state machine for implementing buttons. Connected it to a digital input pin (in *PULLUP* mode) it will handle button presses, releases and holds. You'll need a single Atm_button instance for each button you want to service.

* [begin()](#atm_button--begin-int-attached_pin-presscb_t-press_callback-)
* [onPress]
* [debounce()](#atm_button--debounce-int-delay-)
* [longPress()](#atm_button--longpress-int-max-int-delay-)
* [repeat()](#atm_button--repeat-int-delay-int-speed-)
* [autoPress()](#atm_button--autopress-int-delay-int-press-)
* [onSwitch()](#machine--onswitch-swcb_sym_t-callback-const-char-sym_s-const-char-sym_e-)

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

void setup() {
  btn.begin( 11, btn_change );
  factory.add( btn );
}

void setup() {
  factory.cycle();
}


```

### Atm_button & begin( int attached_pin, presscb_t press_callback ) ###
### Atm_button & begin( int attached_pin ) ###

Initializes an Atm_button object and attaches it to an I/O pin and a callback routine. For every keypress event the callback routine will be called. The I/O pin will be placed in *INPUT_PULLUP* mode so the hardware button connected should connect the pin to ground when pressed.

```c++
Atm_button btn;

void btn_change( int press ) 
{
  if ( press ) {
    led1.toggle( led1.IDLE, led1.START );
  }
}

void setup() {
    btn.begin( 11, btn_change );
}
```

The press argument contains 0 if the event is a button release and 1 if the event is a button press. When in *longpress mode* the press argument may contain other values.

### Atm_button & onPress( Machine * machine, int msg ) ###
### Atm_button & onPress( Machine * machine, int msg_press, int msg_release ) ###
### Atm_button & onPress( presscb_t press_callback ) ###


### Atm_button & debounce( int delay ) ###

Adds a debounce delay of *delay* milliseconds. Default debounce delay is 5 milliseconds.


```c++
void setup() {
    btn.begin( 11, btn_change );
    btn.debounce( 10 );
}
```

### Atm_button & longPress( int max, int delay ) ###

Switches the Atm_button machine to *longpress* mode. In *longpress* mode the events are not triggered by button presses but by button releases. This leads tio a slightly slower response, but the advantage is that we can let one button serve several functions.

The *delay* parameter controls the time the user should keep the button pressed (in millis) for the next event to register. The *max* parameter controls how many events are handled by the button. 

```c++
void setup() {
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
void setup() {
    btn.begin( 11, btn_change );
    btn.repeat();
}
```

The *delay* parameter controls the delay before the first button repeat (default 500 ms). The *repeat* argument controls the time between repeats (default 50).

### Atm_button & autoPress( int delay, int press ) ###

For testing purposes the machine may be configured to automatically generate *press* events with the *autoPress()* method. The *delay* parameter controls the delay between presses. The *press* parameter is passed straight to the callback routine as a parameter.

```c++
void setup() {
    btn.begin( 11, btn_change );
    btn.autoPress( 1000, 1 );
}
```
### Machine & onSwitch( swcb_sym_t callback, const char sym_s[], const char sym_e[] ) ###

To monitor the behavior of this machine you may connect a monitoring function with the Machine::onSwitch() method. 

```c++
void sw_callback( const char label[], const char current[], const char next[], 
      const char trigger[], uint32_t runtime, uint32_t cycles ) {
  Serial.print( millis() );
  Serial.print( " Switching " );
  Serial.print( label );
  Serial.print( " from state " );
  Serial.print( current );
  Serial.print( " to " );
  Serial.print( next );
  Serial.print( " on trigger " );
  Serial.print( trigger );
  Serial.print( " (" );
  Serial.print( cycles );
  Serial.print( " cycles in " );
  Serial.print( runtime );
  Serial.println( " ms)" );
}
```

Use the code below to pass Atm_button's STATES and EVENTS symbol tables to the state machine, open up a serial terminal and watch the machine change states. 

```c++
btn.onSwitch( sw_callback,
    "IDLE\0WAIT\0PRESSED\0REPEAT\0RELEASE\0LIDLE\0LWAIT\0LPRESSED\0LRELEASE\0WRELEASE\0AUTO",
    "EVT_LMODE\0EVT_TIMER\0EVT_DELAY\0EVT_REPEAT\0EVT_PRESS\0EVT_RELEASE\0EVT_COUNTER\0EVT_AUTO\0ELSE" ); 
```



