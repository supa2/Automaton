Welcome to the Automaton wiki!

The following classes and state machines are documented here. The state machines can be used as self contained event-driven components that may be integrated with machines and logic that you build yourself.

### Machine class ###

Base class for defining self contained multi-tasking state machine objects.

### Factory class ###

Add your state machines to the factory class, assign them a priority and run them from the Arduino loop().

### Atm_button class (state machine) ###

A state machine for handling button presses, longpresses, repeats, debouncing, etc.

### Atm_command class (state machine) ###

A state machine that handles commands coming in over a serial line (Stream), parses and interprets them and fires off a handler callback.

### Atm_comparator class (state machine) ###

This state machine monitors an analog input with a configurable sample rate and fires off a callback whenever one of a list of thresholds are crossed. Optionally keeps a running average to smooth out peaks and troughs.

### Atm_fade class (state machine) ###

Control a led via a PWM enabled pin. Control blink speed, pause duration, fade in/out slope and number of repeats.

### Atm_fade class (state machine) ###

Control a led via a digital pin. Control blink speed, pause duration and number of repeats.

### Atm_pulse class (state machine) ###

Monitor digital pin for incoming pulses, fire a callback or send a message on minimum duration.

### Atm_teensywave class (state machine) ###

Generate different waveforms (sine, sawtooth, reverse sawtooth, square, triangle) via the analog out pin of a teensy 3.1. 

### Atm_timer class (state machine) ###

Simple state machine that provides standard timing event functionality. Configure interval and number of repeats. Can be controlled via the message queue. Fires a callback or sends a message via the message queue when the timer triggers.

