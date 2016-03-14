Monitor a digital pin for incoming pulses, fire a callback or send a message when a pulse is detected.

* [begin()](#atm_pulse--begin-int-attached_pin-int-minimum_duration-)
* [onPulse()](#atm_pulse--onpulse-pulsecb_t-press_callback-)
* [onSwitch()](#machine--onswitch-swcb_sym_t-callback-const-char-sym_s-const-char-sym_e-)

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
### Machine & onSwitch( swcb_sym_t callback ) ###

To monitor the behavior of this machine you may connect a monitoring function with the Machine::onSwitch() method. 

```c++
pulse.onSwitch( atm_serial_debug::onSwitch );
```

