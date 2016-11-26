Control a led via a PWM enabled pin. Control blink speed, pause duration, fade in/out slope and number of repeats.

![Fade](images/fade-small.jpg)

<!-- md-tocify-begin -->
* [begin()](#atm_fade--begin-int-attached_pin-)  
* [blink()](#atm_fade--blink-uint32_t-duration-uint32_t-pause_duration-uint16_t-repeat_count--atm_counter_off-)  
* [pause()](#atm_fade--pause-int-duration-)  
* [fade()](#atm_fade--fade-int-fade-)  
* [repeat()](#atm_fade--repeat-int-repeat-)  
* [onFinish()](#atm_fade--onfinish-connector-connector-argument-)  
* [EVT_ON](#evt_on)  
* [EVT_OFF](#evt_off)  
* [EVT_BLINK](#evt_blink)  
* [EVT_START](#evt_start)  
* [EVT_TOGGLE](#evt_toggle)  
* [EVT_TOGGLE_BLINK](#evt_toggle_blink)  
* [trace()](#atm_fade--trace-stream--stream-)  

<!-- md-tocify-end -->

## Synopsis ##

```c++
#include <Automaton.h>

Atm_fade led1, led2;

void setup() {
  led1.begin( 5 ).blink( 40, 250 ).fade( 5 );  // Set up
  led2.begin( 6 ).blink( 40, 50 ).fade( 5 );
  led1.trigger( led1.EVT_BLINK );              // Start fading
  led2.trigger( led2.EVT_BLINK );
}

void loop() {
  automaton.run();
}
```

### Atm_fade & begin( int attached_pin ) ###

Attaches a PWM pin to the Atm_fade machine. Please note that the led will just blink haphazardly on a pin without PWM capability. For the fade effect PWM is required.

```c++
void setup() {
  led1.begin( 5 );
  led1.blink( 40 );
  ...
  led1.state( led1.START );
}
```

Please note that the Atm_fade machine starts up in state IDLE. You can turn the led on by triggering a EVT_ON event or start it blinking with a EVT_BLINK trigger.

```c++
  led1.trigger( led1.EVT_ON );
  led1.trigger( led1.EVT_BLINK );
  led1.trigger( led1.EVT_OFF );
  led1.trigger( led1.EVT_TOGGLE );
  led1.trigger( led1.EVT_TOGGLE_BLINK );
```

### Atm_fade & blink( uint32_t duration, uint32_t pause_duration, uint16_t repeat_count = ATM_COUNTER_OFF ) ###
Alternatively: Atm_fade & blink( uint32_t duration )

Sets the time that the led is fully ON during a cycle in milliseconds.
The three argument version sets the time the led is fully ON, the time the led is fully OFF and the number of repeats.

```c++
void setup() {
  led1.begin( 5 );
  led1.blink( 40 );
  led1.blink( 500, 500 );
  led1.blink( 500, 500, 10 );
  ...
}
```

### Atm_fade & pause( int duration ) ###

Sets the time that the led is fully OFF during a cycle in milliseconds.

```c++
void setup() {
  led1.begin( 5 );
  led1.blink( 40 );
  led1.pause( 100 );
  ...
}
```

### Atm_fade & fade( int fade ) ###

Sets the speed time each fade step takes in milliseconds. The lower this number, the faster the fade effect.

```c++
void setup() {
  led1.begin( 5 ).blink( 50, 100 ).fade( 5 );
  ...
}
```

The fade in (and out) effect is achieved by a slope made up of 32 values. If every step takes 5 ms, the fade cycle will take 160 ms (32 * 5) milliseconds. In the example above that leads to the pattern below.


Off | Fading in | On | Fading out
------------ | ------------- | ------------- | -------------
100 | 160 | 50 | 160

Starting with version 1.0.2 the fade() setting also affects the EVT_ON, EVT_OFF and EVT_TOGGLE events. 

### Atm_fade & repeat( int repeat ) ###

Sets how many times the blink pattern should repeat. Default is *ATM_COUNTER_OFF* which means it will blink indefinitely. Use *1* to blink once, etc...

```c++
void setup() {
  led1.begin( 5 ).blink( 50 ).repeat( 1 ).trigger( led1.EVT_BLINK );
  ...
}
```

The example above gives off a single 50 millisecond pulse and then goes back to sleep (state IDLE).

### Atm_fade & onFinish( {connector}, {connector-argument} ) ###

This method is used to trigger another machine when the current machine's blinking sequence has finished. This can be used to create sequences of blink patterns, but you can also trigger different types of machines in this manner.

```c++
led1.begin( 4 ).blink( 500, 500, 3 ).onFinish( led2, led.EVT_BLINK );
led2.begin( 4 ).blink( 50, 50, 10 );
led1.trigger( led1.EVT_BLINK );
```
The example above will blink a led slowly 3 times and then blink the same led quickly 10 times.

### EVT_ON ###

Turns the led on.

```c++
led.trigger( Atm_fade:EVT_ON );
```

### EVT_OFF ###

Turns the led off.

```c++
led.trigger( Atm_fade:EVT_OFF );
```

### EVT_BLINK ###

Starts the led blinking.

```c++
led.begin( 5 ).blink( 200 );
led.trigger( Atm_fade:EVT_BLINK );
```

### EVT_START ###

Starts the led blinking. (same as EVT_BLINK)

### EVT_TOGGLE ###

Toggle the led on and off.

```c++
led.begin( 5 );
led.trigger( led.EVT_TOGGLE );
```

### EVT_TOGGLE_BLINK ###

Toggle the fading on and off.

```c++
led.begin( 5 ).blink( 100, 900 );
led.trigger( led.EVT_TOGGLE_BLINK );
```

### Atm_fade & trace( Stream & stream ) ###

To monitor the behavior of this machine you may connect a monitoring function with the Machine::trace() method. 

```c++
Serial.begin( 9600 );
led.trace( Serial );
```

**WARNING: This machine changes state 66 times for every fade cycle and will produce a lot of log output quickly**



