Atm_timer implements a timer mechanism as a simple state machine.

* [begin()](#atm_timer--begin-void-)
* [onTimer()](#atm_timer--ontimer-timer_cb_t-timer_callback-)
* [interval()](#atm_timer--interval-int-v-)
* [repeat()](#atm_timer--repeat-int-v-)
* [id()](#atm_timer--id-int-v-)
* [MSG_ON](#msg_on)
* [MSG_OFF](#msg_off)
* [onSwitch()](#machine--onswitch-swcb_sym_t-callback-)

## Synopsis ##

```c++
#include <Automaton.h>
#include <Atm_timer.h>

Atm_timer timer;
Factory factory;

void timer_callback( int id, uint16_t cnt ) 
{
  // Something to do when the timer goes off
}

void setup()
{
  timer.begin();
  timer.interval_seconds( 2 ).repeat( 2 );
  timer.onTimer( timer_callback ).id( 5 );
  factory.add( timer );
}

void loop()
{
  factory.cycle();
}
```

### Atm_timer & begin( [int ms] ) ###

The begin() method has one optional argument, the interval in milliseconds.

```c++
void setup()
{
  timer.begin();
  ...
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

### Atm_timer & interval_millis( int v ) ###

Sets the timer interval, the period after which the timer is fired. The value of *v* is in milliseconds. By default the timer is fired only once after which the timer machine goes to sleep.

```c++
timer.interval( 2000 ); // Wait for 2 seconds
timer.repeat( 2 );
```

Legal values are:

Value | Function
----- | -----
0 | Timer always returns true
1..4294967294 | Returns true if state has been active for at least that many millis
4294967295 | Always returns false (ATM_TIMER_OFF constant)

The interval_millis() method can specify intervals of up to 4294967294 milliseconds (49.7 days)

### Atm_timer & interval_seconds( int v ) ###

Sets the timer interval, the period after which the timer is fired. The value of *v* is in seconds. By default the timer is fired only once after which the timer machine goes to sleep.

```c++
timer.interval_seconds( 2000 ); // Wait for 2000 seconds
timer.repeat( 2 );
```

Legal values are:

Value | Function
----- | -----
0 | Timer always returns true
1..4294967294 | Returns true if state has been active for at least that many seconds
4294967295 | Always returns false (ATM_TIMER_OFF constant)

The interval_seconds() method can specify intervals of up to 4294967294 seconds (about 136.2 years)

### Atm_timer & interval( int v ) ###

Synonym of interval_millis().

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
