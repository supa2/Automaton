A step sequencer that can define up to 8 steps that are processed in sequence whenever an EVT_STEP event is received. Each step can be programmed to generate a callback or to trigger another machine. When only two steps are defined the machine behaves like a flip flop or toggle.

The Atm_step machine is useless on its own, but very useful for linking other machines together.

![Step](images/step-small.jpg)

<!-- md-tocify-begin -->
* [begin()](#atm_step--begin-void-)  
* [onStep()](#atm_step--onstep-int-id-connector-connector-arg-)  
* [EVT_LINEAR](#evt_linear)  
* [EVT_SWEEP](#evt_sweep)  
* [EVT_STEP](#evt_step)  
* [EVT_BACK](#evt_back)  
* [trace()](#atm_step--trace-stream--stream-)  

<!-- md-tocify-end -->

## Synopsis ##

```c++
#include <Automaton.h>

Atm_step step;
Atm_led led1, led2, led3;
Atm_button button;

void setup() {
  led1.begin( 4 ); // Create 3 led machines
  led2.begin( 5 );
  led3.begin( 6 );
  step.begin(); // A step sequencer

  button.begin( 2 )
    .onPress( step, step.EVT_STEP ); // And a button

    step.onStep( 0, led1, led1.EVT_TOGGLE ); // Each step triggers a different led
  step.onStep( 1, led2, led2.EVT_TOGGLE );
  step.onStep( 2, led3, led3.EVT_TOGGLE );
}

void loop() {
  automaton.run();
}
```

Each successive button press advances the step sequencer which will toggle every led on and off in turn.

### Atm_step & begin( void ) ###

Defines a step machine. There are no arguments.

```c++
step.begin();
```

### Atm_step & onStep( int id, {connector}, {connector-arg} ) ###

Registers a callback or registers a trigger to be called when a step becomes active. The callback's v argument contains the id value passed to onStep().

```c++
#include <Automaton.h>

Atm_step step;

void step_callback( int idx, int v, int up ) {
  Serial.print( "Step id=" );
  Serial.println( v );
}

void setup() {
  Serial.begin( 9600 );
  step.begin();
  step.onStep( 0, step_callback );
  step.onStep( 1, step_callback );
  step.onStep( 2, step_callback );
  step.trigger( step.EVT_STEP );
  step.trigger( step.EVT_STEP );
  step.trigger( step.EVT_STEP );
  step.trigger( step.EVT_STEP );
}

void loop() {
  automaton.run();
}
```

This will output:

```
Step id=0
Step id=1
Step id=2
Step id=0
```

Instead of using a callback you can also directly trigger a machine by reference or by label like this:

```c++
step.onStep( 0, led, led.EVT_ON ); // Turn a led on when this step is reached
```

You can empty an existing step by calling step.onStep( id ) with just the id argument;

You can get notification of any step change by using onStep() without the id parameter. This can be combined with onStep() with id parameter.

```c++
#include <Automaton.h>

Atm_step step;
Atm_button button;
Atm_led led[4];

void setup() {
  Serial.begin( 9600 );
  step.begin() 
    .onStep( []( int idx, int v, int up ) {        
      Serial.print( step.state() );
      Serial.print( ": " );
      Serial.println( v );
    });
  for ( int i = 0; i < 4; i++ ) {
    led[i].begin( i + 4 );
    step.onStep( i, led[i], led[i].EVT_TOGGLE );
  }
  button.begin( 2 )
    .onPress( step, step.EVT_STEP );

}

void loop() {
  automaton.run();
}
```

The Atm_led machine already has a toggle event built in, but if it hadn't we could create one with the a two step sequencer connected to a button.

```c++
#include <Automaton.h>

Atm_step toggle;
Atm_led led;
Atm_button button;

void setup() {
  led.begin( 4 );
  toggle.begin() 
    .onStep( 0, led, led.EVT_ON )
    .onStep( 1, led, led.EVT_OFF );
  button.begin( 3 ) 
    .onPress( toggle, toggle.EVT_STEP );
}

void loop() {
  automaton.run();
}
```

Pressing the button connected to pin3 will toggle the led on pin 4 on and off. Oh, and by the way, the bit machine can do a toggle very well too.

### EVT_LINEAR ###

Triggering this event puts the step sequencer in LINEAR mode (default) and resets the state. On the first EVT_STEP trigger received, the machine will switch to step 0.

In linear mode the sequencer will follow this pattern whenever an EVT_STEP is received:

```
0 -> 1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 7 -> 0 -> 1 -> 2 etc...
```

Empty step slots will be skipped.

When an EVT_BACK trigger is received, the direction will be reversed in such a way that the last step is repeated. Modify the example above by changing the setup() method to this:

```c++
void setup() {
  Serial.begin( 9600 );
  step.begin();
  for ( short i = 0; i <= 7; i++ ) {
    step.onStep( i, step_callback ); // Make all steps call the callback
  }
  step.trigger( step.EVT_STEP ); // Step forward
  step.trigger( step.EVT_STEP );
  step.trigger( step.EVT_STEP );
  step.trigger( step.EVT_STEP );
  step.trigger( step.EVT_BACK ); // Backward
  step.trigger( step.EVT_BACK );
  step.trigger( step.EVT_BACK );
  step.trigger( step.EVT_BACK );
  step.trigger( step.EVT_BACK );
  step.trigger( step.EVT_BACK );
  step.trigger( step.EVT_STEP ); // And forward again
  step.trigger( step.EVT_STEP );
  step.trigger( step.EVT_STEP );
}
```

The output will be:

```
Step id=0
Step id=1
Step id=2
Step id=3
Step id=3
Step id=2
Step id=1
Step id=0
Step id=7
Step id=6
Step id=6
Step id=7
Step id=0
```

For an application of this have a look at the analog_bargraph example.

### EVT_SWEEP ###

Triggering this event puts the step sequencer in SWEEP mode and resets the state. On the first EVT_STEP trigger received, the machine will switch to step 0.

In sweep mode the sequencer will follow this pattern whenever an EVT_STEP is received:

```
0 -> 1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 7 -> 6 -> 5 -> 4 -> 3 -> 2 -> 1 -> 0 -> 1 etc...
```

When using a number of steps below 8 make sure the first (0) and last (7) steps are used to create the proper pattern. Empty steps are skipped which leads to double triggers for any steps but the first and last.

See the included knight_rider2 example for an application.

### EVT_STEP ###

This trigger will advance the step sequencer according to the preselected mode.

### EVT_BACK ###

This trigger will reverse the step sequencer when the linear mode is selected. For the other modes EVT_BACK works identical to EVT_STEP.

### Atm_step & trace( Stream & stream ) ###

To monitor the behavior of this machine you may log state change events to a Stream object like Serial. Open the Arduino IDE's serial monitor to see the log messages.

```c++
Serial.begin( 9600 );
step.trace( Serial );
```

