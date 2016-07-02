Control a led via a digital pin. Control blink speed, pause duration and number of repeats. Also very useful for controlling other types of hardware asynchronously, like pulsing buzzers or relays. 

![Led](images/led-small.jpg)

<!-- md-tocify-begin -->
* [begin()](#atm_led--begin-int-attached_pin-bool-activelow--false-)  
* [blink()](#atm_led--blink-int-duration-)  
* [pause()](#atm_led--pause-int-duration-)  
* [fade()](#atm_led--fade-int-fade-)  
* [repeat()](#atm_led--repeat-int-repeat-)  
* [onFinish()](#atm_led--onfinish-machine--next-int-event--evt_start-)  
* [EVT_ON](#evt_on)  
* [EVT_OFF](#evt_off)  
* [EVT_BLINK](#evt_blink)  
* [EVT_TOGGLE](#evt_toggle)  
* [EVT_TOGGLE_BLINK](#evt_toggle_blink)  
* [trace()](#atm_led--trace-stream--stream-)  

<!-- md-tocify-end -->

## Synopsis ##

```c++
#include <Automaton.h>

Atm_led led1, led2;

void setup() {
  led1.begin( 4 ).blink( 40, 250 ); // Setup blinking
  led2.begin( 5 ).blink( 40, 50 ); 
  led1.trigger( led1.EVT_BLINK );   // Start blinking
  led2.trigger( led2.EVT_BLINK );
}

void loop() {
  automaton.run();
}
```

### Atm_led & begin( int attached_pin, bool activeLow = false ) ###

Attaches a digital I/O pin to the Atm_led machine. The pin will be placed in *OUTPUT* mode. Set activeLow to true if you need pull the pin low to activate your led.

This is probably the shortest Automaton sketch (that actually does anything) possible.

```c++
#include <Automaton.h>

Atm_led led;

void setup() {
  led.begin( 4 ).trigger( led.EVT_BLINK );   
}

void loop() {
}
```

If you try it you'll see that the led turns on, but it does not blink. For that to happen the led machine must be constantly cycled:

```c++
#include <Automaton.h>

Atm_led led;

void setup() {
  led.begin( 4 ).trigger( led.EVT_BLINK );   
}

void loop() {
  led.cycle();
}
```

Please note that the Atm_led machine starts up in state *IDLE*. You can turn the led on by sending a EVT_ON message or start it blinking with a EVT_BLINK message. EVT_TOGGLE toggles the led on and off, EVT_TOGGLE_BLINK toggles blinking on and off. These are the available external events:

```c++
  led1.trigger( led1.EVT_ON );
  led1.trigger( led1.EVT_BLINK );
  led1.trigger( led1.EVT_OFF );
  led1.trigger( led1.EVT_TOGGLE );
  led1.trigger( led1.EVT_TOGGLE_BLINK );
```

The begin() method places the machine in the following blink configuration:

```c++
  blink( 500, 500, -1 ); // Blink forever at 1 Herz
```

### Atm_led & blink( int duration ) ###
Alternatively: Atm_fade & blink( uint32_t duration, uint32_t pause_duration, uint16_t repeat_count = ATM_COUNTER_OFF )

Sets the time that the led is fully ON during a cycle in milliseconds. The three argument version sets the time the led is fully ON, the time the led is fully OFF and the number of repeats.

```c++
void setup() {
  led1.begin( 4 );
  led1.blink( 40 ); 
  led1.blink( 40, 100, 10 ); // 40ms on, 100ms off, repeat 10 times
  ...
}
```

### Atm_led & pause( int duration ) ###

Sets the time that the led is fully OFF during a cycle in milliseconds.

```c++
void setup() {
  led1.begin( 4 );
  led1.blink( 40 );
  led1.pause( 100 );
  ...
}
```

### Atm_led & fade( int fade ) ###

This is a dummy method for interface compatibility with the Atm_fade machine. It does nothing here.

### Atm_led & repeat( int repeat ) ###

Sets how many times the blink pattern should repeat. Default is *ATM_COUNTER_OFF* (-1) which means it will blink indefinitely. Use *1* to blink once, etc...

```c++
void setup() {
  led1.begin( 4 ).blink( 40, 100 ).repeat( 1 ).trigger( led1.EVT_BLINK );
  ...
}
```

The example above gives off a single 40 millisecond pulse and then goes back to sleep (state IDLE).

### Atm_led & onFinish( {connector}, {connector-argument} ) ###

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
led.on();

led.trigger( led.EVT_ON );
```

### EVT_OFF ###

Turns the led off.

```c++
led.trigger( led.EVT_OFF );
```

### EVT_BLINK ###

Starts the led blinking.

```c++
led.begin( 5 ).blink( 200 );
led.trigger( led.EVT_BLINK );
```

### EVT_TOGGLE ###

Toggle the led on and off.

```c++
led.begin( 5 );
led.trigger( led.EVT_TOGGLE );
```

### EVT_TOGGLE_BLINK ###

Toggle the blinking on and off.

```c++
led.begin( 5 ).blink( 100, 900 );
led.trigger( led.EVT_TOGGLE_BLINK );
```


### Atm_led & trace( Stream & stream ) ###

To monitor the behavior of this machine you may connect a monitoring function with the Machine::trace() method. 

```c++
Serial.begin( 9600 );
led.trace( Serial );
```

