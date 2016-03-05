A standard Automaton state machine uses about 70 bytes SRAM overhead per state machine instance. In some cases, like when you're trying to run 50 machines on an Arduino Uno or a dozen machines on a tiny AtTiny85 controller that's just too much. For these use cases Automaton Tiny was created. Automaton Tiny consists of a TinyMachine and a TinyFactory class that contain nothing but the minimal functions needed for running state machines. The resulting machine run at about half the SRAM overhead of the standard Machine class based state machines.

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

The naming convention for Tiny machines uses the Att_ prefix instead of the standard Atm_. Replace all mentions of Atm_button by Att_button. The Tiny machine use a 8 bit value to hold the state identifier in contrast to the 16 bit value used by the full size machines;

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

Again replace Atm_ with Att_ and replace 'Machine' with 'TinyMachine' and everything should be fine as long as you haven't use any Machine methods and properties that were dropped for TinyMachine.
