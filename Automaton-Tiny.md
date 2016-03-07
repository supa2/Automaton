### Low memory state machines ###

A standard Automaton state machine uses about 70 bytes SRAM overhead per state machine instance. In some cases, like when you're trying to run 50 machines on an Arduino Uno or a dozen machines on a tiny AtTiny85 controller that's just too much. For these use cases we created *Automaton Tiny*. Automaton Tiny consists of a TinyMachine and a TinyFactory class that contain nothing but the minimal functions needed for running state machines. The resulting machine run at about half the SRAM overhead of the standard Machine class based state machines.

What we left out:

- The debugging code (onSwitch)
- The scheduling priorities
- The machine labels
- The messaging queue

&nbsp; | Machine | TinyMachine
------------ | ------------- | ---------
maximum number of states | 32,768 | 128
begin | Available | Available
msgQueue | Available | -
event | Available | Available 
action | Available | Available
state | Available | Available
trigger | Available | Available
runtime_millis | Available | Available
runtime_micros | Available | Available
asleep | Available | Available
priority | Available | -
cycle | Available | Available
msgWrite | Available | -
msgRead | Available | -
msgPeek | Available | -
msgClear | Available | -
msgMap | Available | -
label | Available | -
onSwitch | Available | -

The TinyFactory class can only manage TinyMachine-based state machines just as the Factory class can only manage Machine-derived state machines.

&nbsp; | Factory | TinyFactory
------------ | ------------- | ---------
add | Available | Available
find | Available | -
cycle | Available | Available 
compatibility | Machine derivations | TinyMachine derivations

It's probably a good strategy to begin by developing a standard *Machine* based machine with the help of the debugging functions and convert to a *TinyMachine* based machine later to squeeze the most out of your microcontroller in a production setup.

### Creating a Tiny Machine ###

To create a Tiny machine base you child class off the *TinyMachine* class instead.

This defines a standard machine:

```c++
class Atm_button : public Machine {

  public:
        Atm_button( void ) : Machine() { class_label = "BTN"; };
```

This defines a *Tiny* machine:

```c++
class Att_button : public TinyMachine {

  public:
        Att_button( void ) : TinyMachine() { };
```

The naming convention for Tiny machines uses the Att_ prefix instead of the standard Atm_. Replace all mentions of Atm_button by Att_button. The Tiny machine use a 8 bit value to hold the state identifier in contrast to the 16 bit value used by the full size machines. (tiny_state_t instead of state_t)

```c++
Atm_button & Atm_button::begin( int attached_pin )
{
        const static state_t state_table[] PROGMEM = {
```

becomes:


```c++
Att_button & Att_button::begin( int attached_pin )
{
        const static tiny_state_t state_table[] PROGMEM = {
```

Again replace Atm_ with Att_ and replace 'Machine' with 'TinyMachine' and everything should be fine as long as you haven't use any Machine-only methods and properties.

### Bundled Tiny state machines ###

The bundled state machines are provided in standard as well as Tiny versions

&nbsp; | Machine | TinyMachine
------------ | ------------- | ---------
Button | Atm_button | Att_button
Serial command processor | Atm_command | Att_command
Analog voltage detector | Atm_comparator | Att_comparator
Blinking led | Atm_led | Att_led
Fading led | Atm_fade | Att_fade
Pulse detection | Atm_pulse | Att_pulse
Waveform generator | Atm_teensywave | Att_teensywave
Timer | Atm_timer | Att_timer

The Att_ versions of these machines lack debugging, messaging and advanced scheduling functions and must be run inside a TinyFactory.

```c++
#include <Automaton.h>
#include <Atm_led.h>

Att_led led;
TinyFactory factory;

void setup() 
{
    led.begin( 4 ).blink( 200 );
    factory.add( led );
    led.trigger( led.EVT_BLINK );
}

void loop() 
{
    factory.cycle();
}
```

This tiny sketch uses 72 of dynamic memory and 2038 bytes of program storage.
The standard sketch uses 138 of dynamic memory and 3318 bytes of program storage.

The examples directory contains a *Tiny* version of the blink sketch called blink_tiny. The blink and blink_tiny sketches illustrate the differences between *Machine* and *TinyMachine* based state machines.


