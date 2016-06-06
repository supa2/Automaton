This is Automaton's built in scheduler. It takes care of multitasking (cycling) Automaton state machines. In normal use you will only see *automaton.run()* in the loop() section of your Arduino sketches.

Normal users will have little interaction with this class.

<!-- md-tocify-begin -->
* [add()](#automaton--add-machine--machine-)  
* [run()](#automaton--run-uint32_t-time--0-)  

<!-- md-tocify-end -->

### Automaton & add( Machine & machine ) ###

Adds a State Machine (component) to the automaton scheduler. 

```c++
#include <Automaton.h>

Atm_led led1;

void setup() {
  automaton.add( led1.begin( 5 ) ); // Add machine explicitly
  led1.trigger( led1.EVT_BLINK ); // Start it blinking
}

void loop() {
  automaton.run();
}
```	
Note that every machine will add itself to the global automaton scheduler automatically when its begin() method is first called, so normally you won't need to use this method and the example above would look like this:

```c++
#include <Automaton.h>

Atm_led led1;

void setup() {
  led1.begin( 5 ); // Initialize led on pin 5
  led1.trigger( led1.EVT_BLINK ); // Start it blinking
}

void loop() {
  automaton.run();
}
```	


### Automaton & run( uint32_t time = 0 ) ###

Executes an Automaton cycle. In a Automaton cycle all machines are cycled once. Normally called from the Arduino loop().

```c++
void loop() {
  automaton.run();
}
```

When the *time* argument is specified and greater than zero, the run() method will cycle until the corresponding number of milliseconds has passed. This can be useful if you want to run one or more state machines in a sequential pattern (normally from the setup method).

```c++
void setup() {
  led1.begin( 5 ).blink( 1000, 1000 ); // Blink a led slowly
  led2.begin( 6 ).blink(  100,  100 ); // Blink a led quickly
  led1.trigger( led1.EVT_BLINK ); // Start them both blinking
  led2.trigger( led2.EVT_BLINK );
  automaton.run( 10000 ); // Let them blink for 10 seconds
  led1.trigger( led1.EVT_OFF ); // Stop the blinking
  led2.trigger( led2.EVT_OFF );
}
```


