Atm_timer implements a timer mechanism as a simple state machine.

* [begin()](#atm_timer--begin-void-)
* [onTimer()](#atm_timer--ontimer-timer_cb_t-timer_callback-)
* [interval()](#atm_timer--interval-int-v-)
* [repeat()](#atm_timer--repeat-int-v-)
* [id()](#atm_timer--id-int-v-)
* [MSG_ON]
* [MSG_OFF]
* [onSwitch()](#machine--onswitch-swcb_sym_t-callback-const-char-sym_s-const-char-sym_e-)

## Synopsis ##

```c++
#include <Automaton.h>
#include <Atm_timer.h>

Atm_timer timer;
Factory factory;

void timer_callback( int id ) 
{
  // Something to do when the timer goes off
}

void setup()
{
  timer.begin();
  timer.interval( 2000 ).repeat( 2 );
  timer.onTimer( timer_callback ).id( 5 );
  factory.add( timer );
}

void loop()
{
  factory.cycle();
}
```

### Atm_timer & begin( void ) ###

The begin() method has no arguments.

```c++
void setup()
{
  timer.begin();
  ...
}
```

### Atm_timer & onTimer( timer_cb_t timer_callback ) ###
### Atm_timer & onTimer( Machine * machine, uint8_t msg ) ###

Registers a callback to be called when the timer expires. Alternatively registers a state machine object to be messaged and the message type to be sent.

```c++
timer.onTimer( timer_callback );

timer.onTimer( &door, door.MSG_OPEN );
```

### Atm_timer & interval( int v ) ###

Sets the timer interval, the period after which the timer is fired. The value of *v* is in milliseconds. By default the timer is fired only once after which the timer machine goes to sleep.

```c++
timer.interval( 2000 );
timer.repeat( 2 );
```

### Atm_timer & repeat( int v ) ###

Sets the number of times the timer will be repeated. Default is 1. If you set this value to *ATM_TIMER_OFF* the timer will repeat indefinitely.

```c++
timer.interval( 2000 );
timer.repeat( 2 );
```
### Atm_timer & id( int v ) ###

Sets the id for the timer. The id is passed to the callback routine so it may be re-used for many diferent timer objects.

```c++
timer.onTimer( timer_callback ).id( 5 );
```
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
cmd.onSwitch( sw, "IDLE\0WAIT\0TRIGGER", "EVT_TIMER\0EVT_COUNTER\0EVT_OFF\0EVT_ON\0ELSE" );
```
