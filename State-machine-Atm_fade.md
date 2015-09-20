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