Welcome to the Automaton wiki!

The following classes and state machines are documented here. The state machines can be used as self contained components that may be integrated with machines and logic that you build yourself.

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

