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

Atm_fade led;
Factory factory;

void setup() 
{
  led.begin( 3 ).blink( 20 ).pause( 50 ).fade( 5 ).state( led.START );
}

void loop() 
{
  factory.cycle()
}
```