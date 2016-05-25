## Tips & tricks ##

### Machines and factories within machines ###

### Poor man's state machine inside a callback ###

### Waiting while a led blinks ###

```cpp
Atm_led led1;

led1.begin( 3 ).blink( 25, 500, 5 ).trigger( led1.EVT_BLINK );
while ( led1.cycle().state() );

```
### Taking timers beyond the 49.7 day millis() rollover limit ###

You can easily combine a timer with a counter to achieve longer delays than the millis maximum of 49.7 days even within one state. A timer expiry switches the state back to itself while the counter is decremented in the ON_ENTER handler. That way the state refreshes its timer long before a second millis() rollover could occur.

To achieve a full year delay you could combine 86,400,000 (number of milliseconds in a day) as a timer value with 365 as a counter value and the machine would stay in that state for (about) a year. 

Such a machine's begin function could look like this:

```c++
static const state_t state_table[] PROGMEM = {
/*             ON_ENTER    ON_LOOP  ON_EXIT EVT_START EVT_TIMER  EVT_COUNTER  ELSE */
/* IDLE   */   ACT_INIT, ATM_SLEEP,      -1,     WAIT,       -1,          -1,   -1, 
/* WAIT   */  ACT_COUNT,        -1,      -1,       -1,     WAIT,      FINISH,   -1, 
/* FINISH */ ACT_FINISH,        -1,      -1,       -1,       -1,          -1, IDLE, 
};
timer.begin( this, 86400000 );
counter.set( 365 );
```
Taking the counter value up to the maximum 65534 would give you about 179 years. 

## Pitfalls ##

### Race conditions ###

