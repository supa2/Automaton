The Automaton timers & counters are implemented as classes with a similar interface. Timers and counters are used a lot in state machines. Machines like Atm_led & Atm_fade run almost entirely on them.

### atm_timer_millis class ###

A millisecond timer that starts running from the moment a state change occurs and expire() at the set() number of milliseconds. The timer will expire immediately when set to zero and will never expire when set to the value *ATM_TIMER_OFF*. 

A timer doesn't need to be reset when a state change occurs. The reset will be automatic because the state timer resets and the timer is relative to the current state timer.

### atm_timer_micros class ###

A microsecond timer that starts running from the moment a state change occurs and expire() at the set() number of microseconds. The timer will expire immediately when set to zero and will never expire when set to the value *ATM_TIMER_OFF*.

A timer doesn't need to be reset when a state change occurs. The reset will be automatic because the state timer resets and the timer is relative to the current state timer.

### atm_counter class ###

An atm_counter object counts down from the set() value to 0.
The counter will ignore decrement() calls and will never expire() when set to the value *ATM_COUNTER_OFF*. Counters are 16 bit unsigned integer values and the highset value is reserved for the *ATM_COUNTER_OFF* flag, so the maximum value that can be counted is 65534.

* [begin()](#begin) (Timers only)
* [set()](#set)
* [decrement()](#decrement) (Counters only)
* [expired()](#expired)


### void begin( Machine * machine, uint32_t v ) ###

(Timers only)

Initializes a timer object by linking it to a Machine object and setting the initial value.

In the Atm_* class definition:
```c++
atm_timer_millis timer1;
atm_timer_micros timer2;
atm_counter counter;
```
In the Atm_*::begin() method:

```c++
timer1.begin( this, 10000 );  // Set timer to a 10 seconds delay
timer2.begin( this, 10000 );  // Set timer to a 10 millisecond delay
counter.set( 10 );
```
The begin() method must be called on a timer object before it's used.

### void set( uint32_t v ) ###

Sets a timer of counter to a value.

```c++
timer1.set( 10000 );  // Set timer to a 10 seconds delay
timer2.set( 10000 );  // Set timer to a 10 millisecond delay

timer1.set( 0 );  // Set timer to expire immediately
timer1.set( ATM_TIMER_OFF );  // Set timer not to expire at all

counter.set( 10 ); // Set counter to expire after 10 decrements
counter.set( 0 ); // Set counter to expire immediately
counter.set( ATM_COUNTER_OFF ); // Set counter not to expire at all

```

To use set() on a timer you must have called begin() first.

###  int expired( void ) ###

Returns 1 if a timer or counter has expired and 0 if it has not. (which happens to be exactly what the event() handler needs)

```c++
  switch ( id ) {
    case EVT_TIMER1 :
      return timer1.expired();
    case EVT_TIMER2 :
      return timer2.expired();
    case EVT_COUNTER :
      return counter.expired();
  }
```

### uint16_t decrement( void ) ###

(Counters only)

Decrements a counter. (typically done inside the action() handler)

```c++
  counter.decrement();
```