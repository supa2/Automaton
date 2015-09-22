
Automaton is an event driven framework that allows you to create Arduino applications that consist entirely of concurrently running state machines interacting with each other. 

**Features**
- Cooperative multi tasking finite state machines base class for building your own state machines 
- State machines are table based, you define the behavior using just a little coding
- Lightweight machine scheduling class with priorities 
- Built in state timers (with milli- and microsecond resolutions) and counters
- Communication between machines via a messaging queue
- Sleep states to save microcontroller cycles
- Pin state monitor
- Debugging (state monitor) hooks (that let you see what the machines are doing)
- Enables modular design and separation of concerns
- State machines can be shared as stand alone Arduino libraries (dependent only on the Automaton library)

Documentation for the (Factory & Machine) base classes and the bundled state machines is linked in the textbox on the right. If you want to make your own machines, look at the machine tutorial.

<tinkerspy@myown.mailcan.com>

```c++
#include <Automaton.h>
#include <Atm_fade.h>
#include <Atm_button.h>

// Four state machines
// Two buttons trigger two fading leds

Atm_fade led1, led2;
Atm_button button1, button2;
Factory factory;

void setup() 
{
  // Initialize two buttons
  button1.begin( 11 );
  button2.begin( 12 );

  // And two fading leds
  led1.begin( 3 ).blink( 500 ).fade( 5 ).repeat( 1 );
  led2.begin( 5 ).blink( 500 ).fade( 5 ).repeat( 1 );

  // Tell the buttons to send a MSG_BLINK message to the leds when they're pressed
  button1.onPress( &led1, led1.MSG_BLINK );    
  button2.onPress( &led2, led2.MSG_BLINK );    

  // Add the machines to a factory
  factory.add (led1 ).add( led2 ).add( button1 ).add( button2 );
}

void loop() 
{
  factory.cycle(); // And run the factory
}
```

### Machine class ###

Base class for defining self contained multi-tasking state machine objects.

### Factory class ###

Add your state machines to the factory class, assign them a priority and run them from the Arduino loop().

## Bundled state machines ##

To get you up and running quickly we've bundled a number of ready to use state machines. Combine them to create your application.

### Atm_button class (state machine) ###

A state machine for handling button presses, longpresses, repeats, debouncing, etc.

### Atm_command class (state machine) ###

A state machine that handles commands coming in over a serial line (Stream), parses and interprets them and fires off a handler callback.

### Atm_comparator class (state machine) ###

This state machine monitors an analog input with a configurable sample rate and fires off a callback whenever one of a list of thresholds are crossed. Optionally keeps a running average to smooth out peaks and troughs.

### Atm_fade class (state machine) ###

Control a led via a PWM enabled pin. Control blink speed, pause duration, fade in/out slope and number of repeats.

### Atm_led class (state machine) ###

Control a led via a digital pin. Control blink speed, pause duration and number of repeats. Can also be used to control other on/off devices like relays and buzzers.

### Atm_pulse class (state machine) ###

Monitor a digital pin for incoming pulses, fire a callback or send a message to another machine.

### Atm_teensywave class (state machine) ###

Generate different waveforms (sine, sawtooth, reverse sawtooth, square, triangle) via the analog out pin of a teensy 3.1. 

### Atm_timer class (state machine) ###

Simple state machine that provides standard timing event functionality. Configure interval and number of repeats. Can be controlled via the message queue. Fires a callback or sends a message via the message queue when the timer triggers.

