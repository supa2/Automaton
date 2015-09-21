Control a led via a digital pin. Control blink speed, pause duration and number of repeats. Also very useful for controlling other types of hardware asynchronously, like pulsing buzzers or relays. Object interface is compatible with Atm_fade.

* [begin()](#atm_led--begin-int-attached_pin-)
* [blink()](#atm_led--blink-int-duration-)
* [pause()](#atm_led--pause-int-duration-)
* [fade()](#atm_led--fade-int-fade-)
* [repeat()](#atm_led--repeat-int-repeat-)
* [onSwitch()](#machine--onswitch-swcb_sym_t-callback-const-char-sym_s-const-char-sym_e-)

## Synopsis ##

```c++
#include <Automaton.h>
#include <Atm_led.h>

Atm_led led1, led2;
Factory factory;

void setup() 
{
  led1.begin( 3 ).blink( 40 ).pause( 250 ).state( led1.START );
  led2.begin( 5 ).blink( 40 ).pause( 50 ).state( led2.START );
  factory.add (led1 ).add( led2 );
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
  led1.state( led1.START );
}
```

Please note that the Atm_led machine starts up in state *IDLE*, this means it won't start until its state is changed to Atm_led::START.

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

### Machine & onSwitch( swcb_sym_t callback, const char sym_s[], const char sym_e[] ) ###

To monitor the behavior of this machine you may connect a monitoring function with the Machine::onSwitch() method. 

```c++
void sw( const char label[], const char current[], const char next[], 
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

Use the code below to pass STATES and EVENTS symbol tables to the state machine, open up a serial terminal and watch the machine change states. 

```c++
cmd.onSwitch( sw, 
  "IDLE\0ON\0START\0BLINK_OFF",
  "EVT_ON_TIMER\0EVT_OFF_TIMER\0EVT_COUNTER\0ELSE" );
```
