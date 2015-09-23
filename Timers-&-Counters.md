The Automaton Timers & Counters are implemented as classes with a similar interface. 

### atm_timer_millis class ###

A millisecond timer that starts running from the moment a state change occurs.
The timer will expire immediately when set to zero, never when set to *ATM_TIMER_OFF*.

### atm_timer_micros class ###

A microsecond timer that starts running from the moment a state change occurs.
The timer will expire immediately when set to zero, never when set to *ATM_TIMER_OFF*.

### atm_counter class ###

An atm_counter object counts down from the set value to 0.
The counter will ignore decrement() calls and never expire() when set to *ATM_COUNTER_OFF*.

* [begin()](#begin) (Timer objects only)
* [set()](#set)
* [decrement()](#decrement) (Counter objects only)
* [expire()](#expire)
