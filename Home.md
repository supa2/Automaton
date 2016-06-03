*Reactive state machine framework for Arduino.*

The Automaton framework allows you to create Arduino applications (appliances) that consist entirely of concurrently running finite state machines (FSM) interacting with one another. Changes are automatically propagated through the appliance like changes in a spreadsheet.

![Sierra Online - Incredible Machine](http://s.emuparadise.org/fup/up/93649-Incredible_Machine_%281993%29%28Sierra_Online%29-1.png)

As in the Incredible Machine game pictured above, Automaton state machines can trigger each other and form intricate control structures. Unlike the game Automaton allows you to create your own state machines and make them interact with the bundled machines and control whatever your Arduino or its peripherals are capable of doing. With Automaton it's easy to build your own incredible machines!

<!-- md-tocify-begin -->
* [Features](#features)  
* [Machine class](#machine-class)  
* [Appliance class](#appliance-class)  
* [Bundled state machines](#bundled-state-machines)  
* [Examples](#examples)  

<!-- md-tocify-end -->

### Features ###
- Cooperative multi tasking state machine base class for building your own state machines 
- State machines are table based, you define the behavior using just a little coding
- Lightweight machine scheduling class 
- Built in state timers and counters
- Communication between machines via event triggers, connectors and direct machine method calls.
- Sleep states to save microcontroller cycles
- Debugging (state monitor) tracing that lets you see what the machines are doing
- Encourages modular design and separation of concerns
- State machines you create can be shared as stand alone Arduino libraries (dependent only on the Automaton library)

Documentation for the (Appliance & Machine) base classes and the bundled state machines is linked in the textbox on the right. If you want to make your own machines, which is where the real fun is, look at the machine building tutorial.

<tinkerspy@myown.mailcan.com>

```c++
#include <Automaton.h>

// A software thermostat monitors a sensor and controls a heater 

Atm_analog thermometer;
Atm_controller thermostat;
Atm_led heater;
Appliance app;

void setup() {
  // Heater controlled by pin 4
  app.component( heater.begin( 4 ) ); 

  // Temperature sensor on analog pin A0
  app.component( thermometer.begin( A0 ) ); 

  // Link them with a controller
  app.component( 
    thermostat.begin()
      .IF( thermometer, '<', 500 )
      .onChange( true, heater, heater.EVT_ON )
      .onChange( false, heater, heater.EVT_OFF )
  );

}

void loop() {
  app.run();
}
```

### Machine class ###

Base class for creating state machines.

### Appliance class ###

Add your state machines to the appliance class (app) and run them from the Arduino loop().

### Bundled state machines ###

To get you up and running quickly we've bundled a number of ready to use state machines. Combine them to create a multitude of different applications.

#### Atm_analog state machine

Monitor an analog pin with optional averaging (low pass filter).

Links: [docs](The-analog-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_analog.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_analog.hpp) 

#### Atm_bit state machine

Logical machine that holds just one value, true or false, 0 or 1, high or low. 
Can trigger other machines on changes and track its status on an indicator led.

Links: [docs](The-bit-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_bit.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_bit.hpp) 

#### Atm_button state machine

A state machine for handling button presses, longpresses, repeats, debouncing, etc.

Links: [docs](The-button-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_button.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_button.hpp) 

#### Atm_command state machine

A state machine that handles commands coming in over a serial line (Stream), parses and interprets them and fires off a handler callback.

Links: [docs](The-command-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_command.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_command.hpp) 

#### Atm_comparator state machine

This state machine monitors an analog input with a configurable sample rate and fires off a callback or triggers (a) machine(s) whenever one of a list of thresholds are crossed. Optionally keeps a moving average to smooth out peaks and troughs.

Links: [docs](The-comparator-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_comparator.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_comparator.hpp) 

#### Atm_controller state machine

Logical machine that holds just one value, true or false, 0 or 1, high or low. 
That value is calculated as a function of the states of machines/callbacks with AND/OR/XOR operators. 
Can trigger other machines on changes and track its status on an indicator led.

Links: [docs](The-controller-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_controller.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_controller.hpp) 

#### Atm_digital state machine

Monitors a digital input pin.
Can trigger other machines on changes and track its status on an indicator led.

Links: [docs](The-digital-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_digital.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_digital.hpp) 

#### Atm_encoder state machine

Use a rotary controller as an input to control other state machines.

Links: [docs](The-encoder-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_encoder.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_encoder.hpp) 

#### Atm_fade state machine

Control a led via a PWM enabled pin. Control blink speed, pause duration, fade in/out slope and number of repeats.

Links: [docs](The-fade-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_fade.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_fade.hpp) 

#### Atm_fan state machine

Turn a single incoming event into multiple outgoing events.

Links: [docs](The-fan machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_fan.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_fan.hpp) 

#### Atm_led state machine

Control a led via a digital pin. Control blink speed, pause duration and number of repeats. Can also be used to control other on/off devices like relays and buzzers. Supports chaining.

Links: [docs](The-led-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_led.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_led.hpp) 


#### Atm_player state machine

Plays musical patterns over a piezo speaker. Can also serve as a generic pattern generator by controlling other machines and processing over the onNote() connectors.

Links: [docs](The-player-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_player.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_player.hpp) 

#### Atm_step state machine

A step sequencer state machine. Create up to 8 steps which all trigger other machines via triggers or callbacks. Step patterns available: linear (default), sweep and burst. The linear pattern can be driven forward or backwards on different triggers. Use two steps in linear mode to create a flip-flop or toggle effect.

Links: [docs](The-step-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_step.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_step.hpp) 

#### Atm_timer state machine

Simple state machine that provides standard timing event functionality. Configure interval and number of repeats. Can be controlled via the message queue. Fires a callback or triggers an event on a different machine when the timer runs out. Create timers of up to 136 years.

Links: [docs](The-timer-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_timer.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_timer.hpp) 

### Examples ###

For running the examples and code snippets on an Arduino Uno connect two buttons to pin 2 & 3 (and GND) and 6 leds to pins 4, ~5, ~6, 7, 8 and ~9 (with a current limiting 330 ohm resistor to GND). 

For the analog_bargraph example connect the Arduino's GND and +5V pins to the outer terminals of a (1K or more) pot meter and pin A0 to the center terminal.

#### blink

The result of the *Machine building tutorial*. A simple do-it-yourself state machine that blinks a led and nothing else. All code is included in one .ino file.


Source code: [blink.ino](/tinkerspy/Automaton/blob/master/examples/blink/blink.ino)

#### blink_modular

The blink example with the state machine neatly separated in .cpp and .h files.

Source code:
[blink_modular.ino](/tinkerspy/Automaton/blob/master/examples/blink_modular/blink_modular.ino),
 [Atm_blink.h](/tinkerspy/Automaton/blob/master/examples/blink_modular/Atm_blink.h),
 [Atm_blink.cpp](/tinkerspy/Automaton/blob/master/examples/blink_modular/Atm_blink.cpp)

#### button

Toggle three leds blinking at different frequencies on and off with one button.

Source code: [button.ino](/tinkerspy/Automaton/blob/master/examples/button/button.ino)

#### fade

Fade two leds on and off at different speeds without using an appliance container.

Source code: [fade.ino](/tinkerspy/Automaton/blob/master/examples/fade/fade.ino)

#### frere_jacques

Play back the french classic 'Frere Jacques' on a piezo speaker on pin 19. Music starts/stops by a button press and 
the playback speed is controlled with an analog pot on A0.

Source code: 
[frere_jacques.ino](/tinkerspy/Automaton/blob/master/examples/frere_jacques/frere_jacques.ino),
[musical_notes.h](/tinkerspy/Automaton/blob/master/examples/frere_jacques/musical_notes.h)

#### knight_rider1

The knight rider sweeping led display using a timer machine that drives a step machine that drives 6 led machines.

Source code: [knight_rider1.ino](/tinkerspy/Automaton/blob/master/examples/knight_rider1/knight_rider1.ino)

#### knight_rider2

The knight rider sweeping led display using a custom made Atm_sweep state machine toggled on and off by a button.

Source code: 
[knight_rider2.ino](/tinkerspy/Automaton/blob/master/examples/knight_rider2/knight_rider2.ino),
 [Atm_sweep.h](/tinkerspy/Automaton/blob/master/examples/knight_rider2/Atm_sweep.h),
 [Atm_sweep.cpp](/tinkerspy/Automaton/blob/master/examples/knight_rider2/Atm_sweep.cpp)

#### led_test

This example shows a button toggling a led while logging all state change information to the serial terminal. Set the Arduino terminal to 9600 baud to monitor the state changes.

Source code: [led_test.ino](/tinkerspy/Automaton/blob/master/examples/led_test/led_test.ino)

#### sos1

Blink a led in a repeating dot-dot-dot-dash-dash-dash-dot-dot-dot pattern. This solution uses one led machine in a sequential pattern by using the timed version of cycle() as a wait/delay statement.

Source code: [sos1.ino](/tinkerspy/Automaton/blob/master/examples/sos1/sos1.ino)

#### sos2

Blink a led in a repeating dot-dot-dot-dash-dash-dash-dot-dot-dot pattern. This solution uses two led machines driving by a step sequencer machine which is in turn driven by a timer machine.

Source code: [sos2.ino](/tinkerspy/Automaton/blob/master/examples/sos2/sos2.ino)

#### sos3

Blink a led in a repeating dot-dot-dot-dash-dash-dash-dot-dot-dot pattern. This solution uses 3 timer machines and 3 led machines chained together in a never ending circle, like snakes biting each others tails.

Source code: [sos3.ino](/tinkerspy/Automaton/blob/master/examples/sos3/sos3.ino)

#### analog_bargraph

An advanced example that uses a combination of comparator, step & led machines to create an interactive bargraph display that changes with the turning of a pot (analog value). 

Source code: [analog_bargraph.ino](/tinkerspy/Automaton/blob/master/examples/analog_bargraph/analog_bargraph.ino)

#### nuclear_missile_launcher

Back in the days of the cold war both sides had intercontinental ballistic missiles aimed at each others cities buried in underground silo's that were manned by soldiers who could launch the missiles in the event of an surprise enemy strike. To avoid the disastrous consequences of a suicidal soldier launching a missile and starting off world war III on his own the launch controls were designed to require at least two different operators to trigger a launch.

In this example we use Automaton to build such a launch trigger mechanism on an Arduino Uno. Two buttons spaced widely apart that must be pressed within two seconds from each other to start a 10 second countdown (LED on pin 8). The countdown will trigger the missile ignition (pin 9).

We used two button machines connected to two bit machines with two timers to reset the bits after 2 second and a controller to check if both bit machines are in the on state. If that is the case the controller starts the countdown which fires the ignition to launch the missile.

Source code: [nuclear_missile_launcher.ino](/tinkerspy/Automaton/blob/master/examples/nuclear_missile_launcher/nuclear_missile_launcher.ino)