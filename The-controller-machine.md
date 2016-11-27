The controller machine can link other machines together in logical machine networks. It's basically a bit machine that changes its state (true or false) in response to the states of other machines. Like a bit machine it serves as a logical gate and can trigger machines or callbacks whenever it changes state. 

Unlike the other bundled machines the controller machine uses *pull* connectors to access the states of the machines it depends on. Every connector can also be a callback which gives complete freedom in formulating logical conditions. The built in pull connectors can be combined with AND, OR and XOR operators.

![Ctrl key](images/ctrl-small.jpg)

<!-- md-tocify-begin -->
* [begin()](#atm_controller--begin-bool-default_state--false-)  
* [onChange()](#atm_controller--onchange-connector-connector-arg-)  
* [onInput()](#atm_controller--oninput-bool-cur_state-connector-connector-arg-)  
* [IF()](#atm_controller-if-connector-char-operator--int-compare_value--0-)  
* [AND()](#atm_controller-and-connector-char-operator--int-compare_value--0-)  
* [OR()](#atm_controller-or-connector-char-operator--int-compare_value--0-)  
* [XOR()](#atm_controller-xor-connector-char-operator--int-compare_value--0-)  
* [led()](#atm_controller--led-int-pin-activelow--false-)  
* [state()](#int-state-void-)  
* [trace()](#atm_controller--trace-stream--stream-)  

<!-- md-tocify-end -->

## Synopsis ##

```c++
#include <Automaton.h>

Atm_analog thermometer;
Atm_controller thermostat;
Atm_led heater;

void setup() {
  heater.begin( 4 ); // Heater controlled by pin 4
  thermometer.begin( A0 ); // Temp sensor on A0
  thermostat.begin()
    .IF( thermometer, '<', 500 )
    .onChange( true, heater, heater.EVT_ON )
    .onChange( false, heater, heater.EVT_OFF );
}

void loop() {
  automaton.run();
}
```

### Atm_controller & begin( bool default_state = false ) ###

Inititalizes the controller machine and sets the default state.

### Atm_controller & onChange( {connector}, {connector-arg} ) ###

Specifies a machine or callback to be triggered whenever the controller machine changes state.

```c++
void setup() {
  ...
  controller.begin()
    .IF( sensor, '+', 500 )
    .onChange( led, led.EVT_TOGGLE );
  ...
}
```

### Atm_controller & onChange( bool new_state, {connector}, {connector-arg} ) ###

Specifies a machine or callback to be triggered whenever the controller machine changes state to *new_state*.

```c++
void setup() {
  ...
  controller.begin()
    .IF( sensor, '+', 500 )
    .onChange( true, led, led.EVT_ON )
    .onChange( false, led, led.EVT_OFF );
  ...
}
```

### Atm_controller & onInput( bool cur_state, {connector}, {connector-arg} ) ###

Specifies a machine or callback to be triggered whenever the controller machine receives an EVT_INPUT event. The machine can be made to respond diferently depending on the current state of the machine. (true or false)

```c++
void setup() {
  ...
  controller.begin()
    .IF( sensor, '+', 500 )
    .onInput( true, led, led.EVT_ON )
    .onInput( false, led, led.EVT_OFF );
  ...
}
```

### Atm_controller IF( {connector}, char operator = '>', int compare_value = 0 ) ###

Specifies the primary controller of a controller machine. There must always be an IF() condition and it must be specified as the first condition. The argument is a connector with its argument. If the connector is a Machine it will retrieve the state() value of that machine. If the connector is a callback, it will retrieve the return value of the callback. The (single character) operator specifies the relational operator to be used. The compare value specifies the value to apply the operator to.

 controller operator | C++ equivalent| Meaning
---- | ---- | ----
= | == | Value equal to
! | != | Value not equal to
> | > | Value greater than
< | < | Value less than
+ | >= | Value greater or equal
- | >= | Value less or equal

Don't forget to use single quotes for the operator character, that's easy to do if you're from a non-C background.

```c++
void setup() {
  ...
  // Becomes true when led.state() > 0
  controller1.begin()
    .IF( led )

  // Becomes true when led.state() = 0
  controller2.begin()
    .IF( led, '=', 0 )

  // Becomes true when sensor.state() >= 500
  controller3.begin()
    .IF( sensor, '+', 500 )
  ...
}
```

### Atm_controller AND( {connector}, char operator = '>', int compare_value = 0 ) ###

Adds an AND condition to the controller machine. You may add a total of one IF condition and 3 logical operators (AND, OR, XOR) to one controller machine.

```c++
void setup() {
  ...
  // Becomes true when all 4 led machines are on
  controller.begin().
    .IF( led1 ).AND( led2 ).AND( led3 ).AND( led4 );
  ...
}
```

### Atm_controller OR( {connector}, char operator = '>', int compare_value = 0 ) ###

Adds an OR condition to the controller machine. You may add a total of one IF condition and 3 logical operators (AND, OR, XOR) to one controller machine.

```c++
void setup() {
  ...
  // Becomes true when anyone of 3 leds are on
  controller.begin()
    .IF( led1 ).OR( led2 ).OR( led3 );
  ...
}
```

### Atm_controller XOR( {connector}, char operator = '>', int compare_value = 0 ) ###

Adds an XOR condition to the controller machine. You may add a total of one IF condition and 3 logical operators (AND, OR, XOR) to one controller machine.

```c++
void setup() {
  ...
  // Becomes true when either, but not both are on
  controller.begin()
    .IF( led1 ).XOR( led2 );
  ...
}
```

XOR conditions are great when you want to control something like a hallway light with more than one switch at the same time. Put the sketch below on an Arduino and it will handle 4 switches for you.

```c++
#include <Automaton.h>

Atm_digital sw1, sw2, sw3, sw4;
Atm_led light;
Atm_controller controller;

void setup() {
  // Any one of 4 switches can switch the light on and off
  // at any time, regardless of the other switches

  light.begin( 4 ); // The ceiling light on pin 4

  sw1.begin( 2, 5, true, true ); // Active low, pullup switches
  sw2.begin( 3, 5, true, true );
  sw3.begin( 6, 5, true, true );
  sw4.begin( 7, 5, true, true );

  controller.begin() // The logical condition that handles it all
    .IF( sw1 ).XOR( sw2 ).XOR( sw3 ).XOR( sw4 )
    .onChange( true, light, light.EVT_ON )
    .onChange( false, light, light.EVT_OFF );
}

void loop() {
  automaton.run();
}
```

An alternative way of doing this with a callback connector (and a lambda function) and the modulo operator for checking for an odd or even result. This way is probably a little faster and it allows you to handle an arbitrary amount of switch inputs in a single controller machine. It's also a little less elegant depending on your taste. 

```c++
#include <Automaton.h>

Atm_digital sw1, sw2, sw3, sw4;
Atm_led light;
Atm_controller controller;

void setup() {
  // Any one of 4 switches can switch the light on and off
  // at any time, regardless of the other switches

  light.begin( 4 ); // The ceiling light on pin 4

  sw1.begin( 2, 5, true, true ); // Active low, pullup switches
  sw2.begin( 3, 5, true, true );
  sw3.begin( 6, 5, true, true );
  sw4.begin( 7, 5, true, true );

  controller.begin() // The logical condition that handles it all
    .IF( []( int idx ) {
      return ( sw1.state() + sw2.state() + sw3.state() + sw4.state() ) % 2 == 0;
    })
    .onChange( true, light, light.EVT_ON )
    .onChange( false, light, light.EVT_OFF );

}

void loop() {
  automaton.run();
}
```

Sometimes it's useful to be able to mix the flexibility of your handcrafted C++ code and existing libraries with Automaton.

### Atm_controller & led( int pin, activeLow = false ) ###

Use the led() method to assign a pin to be used as an state indicator for the controller machine. The controller machine will control the HIGH/LOW state of the pin to match the state of the controller machine. Use it to control a status led that is linked to the state that the controller machine represents. This way you won't need a separate led machine.

```c++
#include <Automaton.h>

Atm_encoder rotaryEncoder;
Atm_controller controller;

void setup() {

  // Define a rotary encoder with range 0..10
  rotaryEncoder.begin( 2, 3 )
    .range( 0, 10 );

  // Controller monitors if encoder > 5
  controller.begin()
    .IF( rotaryEncoder, '>', 5 ) 
    .led( 13 );      

}

void loop() {
  automaton.run();
}

```


### int state( void ) ###

Returns 0 if the condition is in state False (Off/Low/0) and 1 if the controller machine is in state True (On/High/1).

```c++
void setup() {
  bit.begin()
    .trigger( bit.EVT_ON );
  
  controller.begin()
    .IF(  bit )
    .onChange( true, buzzer, buzzer.EVT_ON )
    .onChange( false, buzzer, buzzer.EVT_OFF );
}
```

### Atm_controller & trace( Stream & stream ) ###

To monitor the behavior of this machine you may log state change events to a Stream object like Serial.

```c++
Serial.begin( 9600 );
controller.trace( Serial );
```




