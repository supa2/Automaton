Control a led via a PWM enabled pin. Control blink speed, pause duration, fade in/out slope and number of repeats.

* begin()
* blink()
* pause()
* fade()
* repeat()
* onSwitch()

## Synopsis ##

```c++
#include <Automaton.h>
#include <Atm_fade.h>

Atm_fade led1, led2;
Factory factory;

void setup() 
{
  led1.begin( 3 ).blink( 40 ).pause( 250 ).fade( 5 ).state( led1.START );
  led2.begin( 5 ).blink( 40 ).pause( 50 ).fade( 5 ).state( led2.START );
  factory.add (led1 ).add( led2 );
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

Please note that the Atm_fade machine starts up in state *IDLE*, this means it won't start until its state is changed to Atm_fade::START.

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

### Machine & onSwitch( swcb_sym_t callback, const char sym_s[], const char sym_e[] ) ###

To monitor the behavior of this machine you may connect a monitoring function with the Machine::onSwitch() method. 

```c++
void sw( const char label[], const char current[], const char next[], 
      const char trigger[], uint32_t runtime, uint32_t cycles ) {
  Serial.print( millis() );
  Serial.print( " Switching " );
  Serial.print( label );
  Serial.print( " from state " );
  Serial.print( current );
  Serial.print( " to " );
  Serial.print( next );
  Serial.print( " on trigger " );
  Serial.print( trigger );
  Serial.print( " (" );
  Serial.print( cycles );
  Serial.print( " cycles in " );
  Serial.print( runtime );
  Serial.println( " ms)" );
}
```

Use the code below to pass STATES and EVENTS symbol tables to the state machine, open up a serial terminal and watch the machine change states. 

```c++
cmd.onSwitch( sw, 
  "IDLE\0ON\0START\0STARTU\0UP\0STARTD\0DOWN\0REPEAT",
  "EVT_CNT_FADE\0EVT_TM_FADE\0EVT_TM_ON\0EVT_TM_OFF\0EVT_CNT_RPT\0ELSE" );
```

**WARNING: This machine changes state 66 times for every fade cycle and will produce a lot of log output quickly**



