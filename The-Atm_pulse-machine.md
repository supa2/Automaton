Monitor a digital pin for incoming pulses, fire a callback or send a message when a pulse is detected.

* begin()
* onPulse()
* onSwitch()

## Synopsis ##

```c++
#include <Automaton.h>
#include "Atm_pulse.h"

Atm_pulse pulse;
Factory factory;

void pulse_callback( void ) {

  // Do something when a 20 ms pulse is detected on pin 11
}

void setup()
{
  pulse.begin( 11, 20 ).onPulse( pulse_callback );
  factory.add( pulse );
}

void loop()
{
  factory.cycle();
}
```

### Atm_pulse & begin( int attached_pin, int minimum_duration ) ###

Attaches an Atm_pulse machine to a pin and sets the minimum duration of pulses to watch for.

```c++
pulse.begin( 11, 20 ).onPulse( pulse_callback );
```

### Atm_pulse & onPulse( pulsecb_t press_callback ) ###
### Atm_pulse & onPulse( Machine * machine, uint8_t msg ) ###

Registers a callback that gets called when a pulse is detected.

```c++
pulse.begin( 11, 20 );
pulse.onPulse( pulse_callback );
```

Or registers a state machine that gets a message when a pulse is detected.

```c++
pulse.begin( 11, 20 );
pulse.onPulse( &door, MSG_OPEN );
```

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
  "IDLE\0WAIT\0PULSE",
  "EVT_TIMER\0EVT_HIGH\0EVT_LOW\0ELSE" );
```
