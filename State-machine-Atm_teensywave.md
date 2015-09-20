Generate different waveforms (sine, sawtooth, reverse sawtooth, square, triangle) via the analog out pin of a teensy 3.1. This machine is only compatible with the Teensy 3.1 which has a true analog output pin: A14. The code is partially based on Sine wave example by Paul Stoffregen (maker of Teensy).

* begin()
* onSwitch()
* MSG_TOGGLE

## Waveforms ## 

Index | Waveform
------------ | -------------
1 | Sine wave
2 | Sawtooth wave
3 | Reverse Sawtooth wave
4 | Triangle wave
5 | Square wave

Synopsis

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
  "IDLE\0START_SN\0SINE\0START_SW\0SAW\0START_SR\0SAWR"
  "\0START_TR\0TRI\0START_SQ\0SQON\0SQOFF",
  "EVT_COUNTER\0EVT_TIMER\0EVT_TOGGLE\0ELSE" );
```

