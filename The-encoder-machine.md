The encoder state machine monitors a 2 pin (+GND) rotary encoder for changes. On every movement of the axis either an up or down connector is called. The encoder can be linked to a numeric range in which case the current value can be accessed via the state() method or referenced by a condition machine. The encoder machine is in many ways interchangeable with the analog machine.

Developed for and tested with the [Sparkfun rotary encoder](https://www.sparkfun.com/products/9117).

![Rotary encoder](images/rotary-small.jpg)

<!-- md-tocify-begin -->
* [begin()](#atm_encoder--begin-int-pin1-int-pin2-int-divider--1-)  
* [onChange()](#atm_encoder--onchange-connector-connector-arg-)  
* [set()](#atm_encoder--set-int-v-)  
* [range()](#atm_encoder--range-int-tolow-int-tohigh-bool-wrap--false-)  
* [state()](#int-state-void-)  
* [trace()](#atm_encoder--trace-stream--stream-)  

<!-- md-tocify-end -->

## Synopsis ##

```c++
#include <Automaton.h>

Atm_led led;
Atm_encoder frequency;

void setup() {

  led.begin( 4 ) .led on pin 4
    .start(); // Blink repeatedly

  frequency.begin( 2, 3 ) // Set frequency with encoder on pins 2 & 3
    .range( 1, 50 ) // 1 to 50 Hz
    .onChange( [] ( int idx, int v, int up ) {
       led.frequency( v );   
    });

}

void loop() {
  automaton.run();
}
```

### Atm_encoder & begin( int pin1, int pin2, int divider = 1 ) ###

Inititalizes the rotary encoder machine and sets the control pins.
Optionally sets a divider to slow the encoder down by discarding events.



### Atm_encoder & onChange( {connector}, {connector-arg} ) ###

Specifies a machine or callback to be triggered whenever the encoder machine changes position.

```c++
void setup() {
  ...
  encoder.begin( 2, 3 )
    .onChange( led, led.EVT_BLINK );
  ...
}
```


### Atm_encoder & onChange( bool status, {connector}, {connector-arg} ) ###

Specifies a machine or callback to be triggered whenever the encoder machine changes position in the clockwise (status true or ATM_UP) or counterclockwise (status false or ATM_DOWN) direction.

```c++
void setup() {
  ...
  encoder.begin( 2, 3 )
    .onChange( ATM_UP, led, led.EVT_BLINK )
    .onChange( ATM_DOWN, led, led.EVT_BLINK );
  ...
}
```
### Atm_encoder & set( int v ) ###

Sets the internal position counter. It's your responsibility to make sure you set it within the limits set by range() method.

```c++
void setup() {
  encoder.begin( 2, 3 ) 
    .range( -10, 10 )
    .set( 0 );
}
```

### Atm_encoder & range( int toLow, int toHigh, bool wrap = false ) ###

Sets the range over which the internal position counter may move. If the current position counter is outside the set range it will be set to the *toLow* value. The position counter starts at 0 when the machine is first initialized.

If the wrap parameter is set to true the encoder will wrap past the end or beginning of the range.

```c++
void setup() {
  encoder.begin( 2, 3 ) 
    .range( -10, 10 )
    .set( 0 );
}
```

### int state( void ) ###

Returns the integer value of the internal position counter.

```c++
#include <Automaton.h>

Atm_encoder encoder;
Atm_controller controller;
Atm_led led;

void setup() {
  Serial.begin( 9600 );
  // An encoder that prints its position to the serial monitor
  encoder.begin( 2, 3 ) 
    .range( 0, 10 )
    .onChange( [] ( int idx, int v, int up ) {
      Serial.print( up ? "Encoder moved up to: " : "Encoder moved down to: " );
      Serial.println( encoder.state() ); // Uses state()        
    });
  // And a controller that turns a led on if the encoder turns past position 5
  controller.begin()
    .IF( encoder, '>', 5  )  // Implicitly uses state()
    .led( 4 );
}

void loop() {
  automaton.run();
}
```
In the callback function the v argument contains the same value as state().

### Atm_encoder & trace( Stream & stream ) ###

To monitor the behavior of this machine you may log state change events to a Stream object like Serial.

```c++
Serial.begin( 9600 );
encoder.trace( Serial );
```
