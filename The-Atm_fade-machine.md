Control a led via a PWM enabled pin. Control blink speed, pause duration, fade in/out slope and number of repeats.

* [begin()](#atm_fade--begin-int-attached_pin-)
* [blink()](#atm_fade--blink-int-duration-)
* [pause()](#atm_fade--pause-int-duration-)
* [fade()](#atm_fade--fade-int-fade-)
* [repeat()](#atm_fade--repeat-int-repeat-)
* [MSG_ON](#msg_on)
* [MSG_OFF](#msg_off)
* [MSG_BLINK](#msg_blink)
* [onSwitch()](#machine--onswitch-swcb_sym_t-callback-)

## Synopsis ##

```c++
#include <Automaton.h>
#include <Atm_fade.h>

Atm_fade led1, led2;
Factory factory;

void setup() 
{
  led1.begin( 3 ).blink( 40 ).pause( 250 ).fade( 5 );  // Set up
  led2.begin( 5 ).blink( 40 ).pause( 50 ).fade( 5 );
  factory.add (led1 ).add( led2 );                     // Add to a factory
  led1.msgWrite( led1.MSG_BLINK );                     // Start fading
  led2.msgWrite( led2.MSG_BLINK );
}

void loop() 
{
  factory.cycle();
}
```

### Atm_fade & begin( int attached_pin ) ###

Attaches a PWM pin to the Atm_fade machine. Please note that the led will just blink haphazardly on a pin without PWM capability. For the fade effect PWM is required.

```c++
void setup() 
{
  led1.begin( 3 );
  led1.blink( 40 );
  ...
  led1.state( led1.START );
}
```

Please note that the Atm_fade machine starts up in state IDLE. You can turn the led on by sending a MSG_ON message or start it blinking with a MSG_BLINK message.

  led1.msgWrite( led1.MSG_ON );
  led1.msgWrite( led1.MSG_BLINK );
  led1.msgWrite( led1.MSG_OFF );

### Atm_fade & blink( int duration ) ###

Sets the time that the led is fully ON during a cycle in milliseconds.

```c++
void setup() 
{
  led1.begin( 3 );
  led1.blink( 40 );
  ...
}
```

### Atm_fade & pause( int duration ) ###

Sets the time that the led is fully OFF during a cycle in milliseconds.

```c++
void setup() 
{
  led1.begin( 3 );
  led1.blink( 40 );
  led1.pause( 100 );
  ...
}
```

### Atm_fade & fade( int fade ) ###

Sets the speed time each fade step takes in milliseconds. The lower this number, the faster the fade effect.

```c++
void setup() 
{
  led1.begin( 3 );
  led1.blink( 50 );
  led1.pause( 100 );
  led1.fade( 5 );
  ...
}
```

The fade in (and out) effect is achieved by a slope made up of 32 values. If every step takes 5 ms, the fade cycle will take 160 ms (32 * 5) milliseconds. In the example above that leads to the pattern below.


Off | Fading in | On | Fading out
------------ | ------------- | ------------- | -------------
100 | 160 | 50 | 160

### Atm_fade & repeat( int repeat ) ###

Sets how many times the blink pattern should repeat. Default is *ATM_COUNTER_OFF* which means it willblink indefinitely. Use *1* to blink once, etc...

```c++
void setup() 
{
  led1.begin( 3 ).blink( 40 ).repeat( 1 ).state( led1.START );
  ...
}
```

The example above gives off a single 50 millisecond pulse and then goes back to sleep (state IDLE).


### MSG_ON ###

Turns the led on.

```c++
led.msgWrite( MSG_ON );
```

### MSG_OFF ###

Turns the led off.

```c++
led.msgWrite( MSG_OFF );
```

### MSG_BLINK ###

Starts the led blinking.

```c++
led.begin( 3 ).blink( 200 ).repeat( 1 );
led.msgWrite( MSG_BLINK, 3 );
```

Sending three MSG_BLINK messages to the led object makes the led blink three times (since repeat = 1).

### Machine & onSwitch( swcb_sym_t callback ) ###

To monitor the behavior of this machine you may connect a monitoring function with the Machine::onSwitch() method. 

```c++
led.onSwitch( atm_serial_debug::onSwitch );
```

**WARNING: This machine changes state 66 times for every fade cycle and will produce a lot of log output quickly**



