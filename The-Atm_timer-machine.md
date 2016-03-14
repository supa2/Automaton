Atm_timer implements a timer mechanism as a simple state machine.

* [begin()](#atm_timer--begin-void-)
* [onTimer()](#atm_timer--ontimer-timer_cb_t-timer_callback-)
* [interval()](#atm_timer--interval-int-v-)
* [repeat()](#atm_timer--repeat-int-v-)
* [id()](#atm_timer--id-int-v-)
* [MSG_ON](#msg_on)
* [MSG_OFF](#msg_off)
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

Sets the number of times the timer will be repeated. Default is 1. If you set this value to *ATM_COUNTER_OFF* the timer will repeat indefinitely.

```c++
timer.interval( 2000 );
timer.repeat( 2 );
```
### Atm_timer & id( int v ) ###

Sets the id for the timer. The id is passed to the callback routine so it may be re-used for many diferent timer objects.

```c++
timer.onTimer( timer_callback ).id( 5 );
```

### MSG_ON ###

Turns an idle timer on.

```c++
timer.msgWrite( MSG_ON );
```

### MSG_OFF ###

Turns a running timer off.

```c++
timer.msgWrite( MSG_OFF );
```
### Machine & onSwitch( swcb_sym_t callback ) ###

To monitor the behavior of this machine you may connect a monitoring function with the Machine::onSwitch() method. 

```c++
timer.onSwitch( atm_serial_debug::onSwitch );
```
