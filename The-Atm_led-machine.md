Control a led via a digital pin. Control blink speed, pause duration and number of repeats. Also very useful for controlling other types of hardware asynchronously, like pulsing buzzers or relays. Object interface is compatible with Atm_fade.

* [begin()](#atm_led--begin-int-attached_pin-)
* [blink()](#atm_led--blink-int-duration-)
* [pause()](#atm_led--pause-int-duration-)
* [fade()](#atm_led--fade-int-fade-)
* [repeat()](#atm_led--repeat-int-repeat-)
* [MSG_ON](#msg_on)
* [MSG_OFF](#msg_off)
* [MSG_BLINK](#msg_blink)
* [onSwitch()](#machine--onswitch-swcb_sym_t-callback-)

## Synopsis ##

```c++
#include <Automaton.h>
#include <Atm_led.h>

Atm_led led1, led2;
Factory factory;

void setup() 
{
  led1.begin( 3 ).blink( 40 ).pause( 250 ); // Setup 
  led2.begin( 5 ).blink( 40 ).pause( 50 ); 
  factory.add (led1 ).add( led2 );          // Add to a factory
  led1.msgWrite( led1.MSG_BLINK );          // Start blinking
  led2.msgWrite( led2.MSG_BLINK );
}

void loop() 
{
  factory.cycle();
}
```

### Atm_led & begin( int attached_pin ) ###

Attaches a digital I/O pin to the Atm_led machine. The pin will be placed in *OUTPUT* mode.

```c++
void setup() 
{
  led1.begin( 3 );
  led1.blink( 40 );
  ...
  led1.msgWrite( led1.MSG_BLINK );
}
```

Please note that the Atm_led machine starts up in state *IDLE*. You can turn the led on by sending a MSG_ON message or start it blinking with a MSG_BLINK message.

```c++
  led1.msgWrite( led1.MSG_ON );
  led1.msgWrite( led1.MSG_BLINK );
  led1.msgWrite( led1.MSG_OFF );
```

### Atm_led & blink( int duration ) ###

Sets the time that the led is fully ON during a cycle in milliseconds.

```c++
void setup() 
{
  led1.begin( 3 );
  led1.blink( 40 );
  ...
}
```

### Atm_led & pause( int duration ) ###

Sets the time that the led is fully OFF during a cycle in milliseconds.

```c++
void setup() 
{
  led1.begin( 3 );
  led1.blink( 40 );
  led1.pause( 100 );
  ...
}
```

### Atm_led & fade( int fade ) ###

This is a dummy method for interface compatibility with the Atm_fade machine. It does nothing here.

### Atm_led & repeat( int repeat ) ###

Sets how many times the blink pattern should repeat. Default is *ATM_COUNTER_OFF* which means it will blink indefinitely. Use *1* to blink once, etc...

```c++
void setup() 
{
  led1.begin( 3 ).blink( 40 ).repeat( 1 ).state( led1.START );
  ...
}
```

The example above gives off a single 50 millisecond pulse and then goes back to sleep (state IDLE).

### MSG_ON ###

Turns the led on.

```c++
led.msgWrite( MSG_ON );
```

### MSG_OFF ###

Turns the led off.

```c++
led.msgWrite( MSG_OFF );
```

### MSG_BLINK ###

Starts the led blinking.

```c++
led.begin( 3 ).blink( 200 ).repeat( 1 );
led.msgWrite( MSG_BLINK, 3 );
```

Sending three MSG_BLINK messages to the led object makes the led blink three times (since repeat = 1).

### Machine & onSwitch( swcb_sym_t callback ) ###

To monitor the behavior of this machine you may connect a monitoring function with the Machine::onSwitch() method. 

```c++
led.onSwitch( atm_serial_debug::onSwitch );
```
