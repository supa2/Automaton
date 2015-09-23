The Automaton timers & counters are implemented as classes with a similar interface. Timers and counters are used a lot in state machines. Machines like Atm_led & Atm_fade run almost entirely on them.

### atm_timer_millis class ###

A millisecond timer that starts running from the moment a state change occurs and expire() at the set() number of milliseconds. The timer will expire immediately when set to zero, never when set to the value *ATM_TIMER_OFF*. 

A timer doesn't need to be reset when a state change occurs. The reset will be automatic because the state timer resets and the timer is relative to the current state timer.

### atm_timer_micros class ###

A microsecond timer that starts running from the moment a state change occurs and expire() at the set() number of microseconds. The timer will expire immediately when set to zero, never when set to the value *ATM_TIMER_OFF*.

A timer doesn't need to be reset when a state change occurs. The reset will be automatic because the state timer resets and the timer is relative to the current state timer.

### atm_counter class ###

An atm_counter object counts down from the set() value to 0.
The counter will ignore decrement() calls and never expire() when set to the value *ATM_COUNTER_OFF*.

* [begin()](#begin) (Timers only)
* [set()](#set)
* [decrement()](#decrement) (Counters only)
* [expire()](#expire)
