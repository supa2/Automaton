This state machine generates different waveforms (sine, sawtooth, reverse sawtooth, square, triangle) via the analog out pin of a teensy 3.1. This machine is only compatible with the Teensy 3.1 and LC which have a true analog output pin. The code is partially based on sine wave example code by Paul Stoffregen (maker of Teensy).

* [begin()](#atm_teensywave--begin-int-attached_pin-int-steps-int-delay-)
* [MSG_TOGGLE](#msg_toggle)
* [onSwitch()](#machine--onswitch-swcb_sym_t-callback-const-char-sym_s-const-char-sym_e-)

The output of a teensy 3.1 running teensywave on a scope:

[[images/scope.png]]


## Waveforms ##

Index | Waveform
------------ | -------------
1 | Sine wave
2 | Sawtooth wave
3 | Reverse Sawtooth wave
4 | Triangle wave
5 | Square wave

## Synopsis ##

```c++
#include <Automaton.h>
#include <Atm_teensywave.h>
#include <Atm_button.h>

int pinOut = A14; // Teensy 3.1 analog output pin (LC uses A12)
int pinIn = 11;   // Button attached to pin 11 (pullup)

Atm_teensywave wave;
Atm_button btn;
Factory factory;

void btnHandler( int press ) {
  if ( press ) {
    wave.msgWrite( wave.MSG_TOGGLE );
  }
}

void setup()
{
  wave.begin( pinOut, 314, 50 ); 
  btn.begin( pinIn, btnHandler );
  factory.add( wave ).add( btn );
}

void loop()
{
  factory.cycle();
}
```

### Atm_teensywave & begin( int attached_pin, int steps, int delay ) ###

Connects the machine to an analog output pin (A14) and starts outputting a wave. You may control the frequency of the wave by changing the number of steps per period and the delay between steps (in microseconds). Note that the wave get very choppy once you proceed above 100 Hz (with the exception of the square wave).

```c++
void setup()
{
  wave.begin( A14, 314, 50 ); 
}
```

### MSG_TOGGLE ###

The machine will switch from one waveform to another on receipt of a MSG_TOGGLE message.

```c++
wave.msgWrite( MSG_TOGGLE );
```

### Machine & onSwitch( swcb_sym_t callback ) ###

To monitor the behavior of this machine you may connect a monitoring function with the Machine::onSwitch() method. 

```c++
btn.onSwitch( atm_serial_debug::onSwitch );
```
