## Tips & tricks ##

### Sending and receiving messages with data ###
### Machines and factories within machines ###
### Simulating priorities with TinyFactory ###

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

### Hanging messages in the message queue ###
### Forgetting to use begin() on a timer ###

In this case expire() will probably always return true and the timer will expire immediately.

### Start up race condition with machine->trigger( EVT_XXX ) ###

```c++
void setup() {
  led.begin( 3 );
  led.cycle().trigger( EVT_ON );
}

void loop() {
  led.cycle();
}


```
