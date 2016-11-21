*Reactive state machine framework for Arduino.*

The Automaton framework allows you to create Arduino applications that consist entirely of concurrently running components (finite state machines)
interacting with one another. Changes are automatically propagated through the application like changes in a spreadsheet.

Automaton components can trigger each other and form intricate control structures. Automaton helps you create your own components, makes them interact 
with the bundled components to let you control whatever your Arduino or its peripherals are capable of doing.

<!-- md-tocify-begin -->
* [Features](#features)  
* [Machine class](#machine-class)  
* [Bundled components](#bundled-components)  
* [Examples](#examples)  

<!-- md-tocify-end -->

### Features ###
- Cooperative multi tasking state machine base class for building your own components 
- Components are table based state machines, you define the behavior using just a little coding
- Lightweight machine scheduling class 
- Built in state timers and counters
- Communication between components via event triggers, connectors and direct method calls.
- Sleep states to save microcontroller cycles
- Debugging (state monitor) tracing that lets you see what the components are doing
- Encourages modular design and separation of concerns
- Components you create can be shared as stand alone Arduino libraries (dependent only on the Automaton library)

Documentation for the (Automaton & Machine) base classes and the bundled components is linked in the textbox on the right. 
If you want to make your own components, which is where the real fun is, look at the machine building tutorial.

<tinkerspy@myown.mailcan.com>

```c++
#include <Automaton.h>

// Toggle a blinking led with a button

int ledPin = 5;
int buttonPin = 2;

Atm_led led;
Atm_button button;

void setup() {

  led.begin( ledPin )
    .blink( 200, 200 ); // Set up a led to blink 200ms/200ms

  button.begin( buttonPin )
    .onPress( led, led.EVT_TOGGLE_BLINK ); // Toggle the led when button pressed

}

void loop() {
  automaton.run();
}
```

### Machine class ###

Base class for creating state machines (components).

### Bundled components ###

To get you up and running quickly we've bundled a number of ready to use components. Combine them to create a multitude of different applications.

#### Atm_analog component

Monitor an analog pin with optional averaging (low pass filter).

Links: [docs](The-analog-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_analog.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_analog.hpp) 

#### Atm_bit component

Logical component that holds just one value, true or false, 0 or 1, high or low. 
Can trigger other components on changes and track its status on an indicator led.

Links: [docs](The-bit-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_bit.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_bit.hpp) 

#### Atm_button component

A component for handling button presses, longpresses, repeats, debouncing, etc.

Links: [docs](The-button-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_button.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_button.hpp) 

#### Atm_command component

A component that handles commands coming in over a serial line (Stream), parses and interprets them and fires off a handler callback.

Links: [docs](The-command-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_command.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_command.hpp) 

#### Atm_comparator component

This component monitors an analog input with a configurable sample rate and fires off a callback or triggers (a) component(s) whenever one of a list of thresholds are crossed. 
Optionally keeps a moving average to smooth out peaks and troughs.

Links: [docs](The-comparator-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_comparator.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_comparator.hpp) 

#### Atm_controller component

Logical component that holds just one value, true or false, 0 or 1, high or low. 
That value is calculated as a function of the states of component/callbacks with AND/OR/XOR operators. 
Can trigger other components on changes and track its status on an indicator led.

Links: [docs](The-controller-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_controller.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_controller.hpp) 

#### Atm_digital component

Monitors a digital input pin.
Can trigger other components on changes and track its status on an indicator led.

Links: [docs](The-digital-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_digital.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_digital.hpp) 

#### Atm_encoder component

Use a rotary controller as an input to control other component.

Links: [docs](The-encoder-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_encoder.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_encoder.hpp) 

#### Atm_fade component

Control a led via a PWM enabled pin. Control blink speed, pause duration, fade in/out slope and number of repeats.

Links: [docs](The-fade-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_fade.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_fade.hpp) 

#### Atm_fan component

Turn a single incoming event into multiple outgoing events.

Links: [docs](The-fan machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_fan.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_fan.hpp) 

#### Atm_led component

Control a led via a digital pin. Control blink speed, pause duration and number of repeats. Can also be used to control other on/off devices like relays and buzzers. Supports chaining.

Links: [docs](The-led-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_led.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_led.hpp) 


#### Atm_player component

Plays musical patterns over a piezo speaker. Can also serve as a generic pattern generator by controlling other machines and processing over the onNote() connectors.

Links: [docs](The-player-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_player.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_player.hpp) 

#### Atm_servo component

Controls one or more servo's. 

Links: [docs](The-servo-machine),
 [cpp](/tinkerspy/Automaton-Servo/blob/master/src/Atm_servo.cpp),
 [h](/tinkerspy/Automaton-Servo/blob/master/src/Atm_servo.h)

#### Atm_step component

A step sequencer component. Create up to 8 steps which all trigger other components via triggers or callbacks. 
Step patterns available: linear (default) and sweep. The linear pattern can be driven forward or backwards on different triggers. 
Use two steps in linear mode to create a flip-flop or toggle effect.

Links: [docs](The-step-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_step.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_step.hpp) 

#### Atm_timer component

Simple component that provides standard timing event functionality. Configure interval and number of repeats. Can be controlled via events. Fires a callback or triggers an event on a different component when the timer runs out. Create timers of up to 136 years.

Links: [docs](The-timer-machine),
 [cpp](/tinkerspy/Automaton/blob/master/src/Atm_timer.cpp),
 [hpp](/tinkerspy/Automaton/blob/master/src/Atm_timer.hpp) 

### Examples ###

For running the examples and code snippets on an Arduino Uno connect two buttons to pin 2 & 3 (and GND) and 6 leds to pins 4, ~5, ~6, 7, 8 and ~9 (with a current limiting 330 ohm resistor to GND). 

For the led_fuel_gauge example connect the Arduino's GND and +5V pins to the outer terminals of a (1K or more) pot meter and pin A0 to the center terminal.

#### Blink

The result of the *Machine building tutorial*. A simple do-it-yourself component that blinks a led and nothing else. All code is included in one .ino file.


Source code: [blink.ino](/tinkerspy/Automaton/blob/master/examples/blink/blink.ino)

#### Blink Modular

The blink example with the state machine component neatly separated in .cpp and .h files.

Source code:
[blink_modular.ino](/tinkerspy/Automaton/blob/master/examples/blink_modular/blink_modular.ino),
 [Atm_blink.h](/tinkerspy/Automaton/blob/master/examples/blink_modular/Atm_blink.h),
 [Atm_blink.cpp](/tinkerspy/Automaton/blob/master/examples/blink_modular/Atm_blink.cpp)

#### Button

Toggle three leds blinking at different frequencies on and off with one button.

Source code: [button.ino](/tinkerspy/Automaton/blob/master/examples/button/button.ino)

#### Fade

Fade two leds on and off at different speeds without using the automaton scheduler object.

Source code: [fade.ino](/tinkerspy/Automaton/blob/master/examples/fade/fade.ino)

#### Frere Jacques

Play back the french classic 'Frere Jacques' on a piezo speaker on pin 19. Music starts/stops by a button press and 
the playback speed is controlled with an analog pot on A0.

Source code: 
[frere_jacques.ino](/tinkerspy/Automaton/blob/master/examples/frere_jacques/frere_jacques.ino),
[musical_notes.h](/tinkerspy/Automaton/blob/master/examples/frere_jacques/musical_notes.h)

#### Knight Rider 1

The knight rider sweeping led display using a timer machine that drives a step machine that drives 6 led machines.

Source code: [knight_rider1.ino](/tinkerspy/Automaton/blob/master/examples/knight_rider1/knight_rider1.ino)

#### Knight Rider 2

The knight rider sweeping led display using a custom made Atm_sweep component toggled on and off by a button.

Source code: 
[knight_rider2.ino](/tinkerspy/Automaton/blob/master/examples/knight_rider2/knight_rider2.ino),
 [Atm_sweep.h](/tinkerspy/Automaton/blob/master/examples/knight_rider2/Atm_sweep.h),
 [Atm_sweep.cpp](/tinkerspy/Automaton/blob/master/examples/knight_rider2/Atm_sweep.cpp)

#### Knight Rider 3

The knight rider sweeping led display using the Atm_player component as a generic pattern generator.

Source code: 
[knight_rider3.ino](/tinkerspy/Automaton/blob/master/examples/knight_rider3/knight_rider3.ino)

#### LED Test

This example shows a button toggling a led while logging all state change information to the serial terminal. 
Set the Arduino terminal to 9600 baud to monitor the state changes.

Source code: [led_test.ino](/tinkerspy/Automaton/blob/master/examples/led_test/led_test.ino)

#### S.O.S. 1

Blink a led in a repeating dot-dot-dot-dash-dash-dash-dot-dot-dot pattern. This solution uses one led component in a sequential pattern by using the timed version of cycle() as a wait/delay statement.

Source code: [sos1.ino](/tinkerspy/Automaton/blob/master/examples/sos1/sos1.ino)

#### S.O.S. 2

Blink a led in a repeating dot-dot-dot-dash-dash-dash-dot-dot-dot pattern. This solution uses two led components driven by a step sequencer which is in turn driven by a timer component.

Source code: [sos2.ino](/tinkerspy/Automaton/blob/master/examples/sos2/sos2.ino)

#### S.O.S. 3

Blink a led in a repeating dot-dot-dot-dash-dash-dash-dot-dot-dot pattern. This solution uses 3 timer components and 3 led components chained together in a never ending circle, like snakes biting each others tails.

Source code: [sos3.ino](/tinkerspy/Automaton/blob/master/examples/sos3/sos3.ino)

#### LED Fuel Gauge

An advanced example that uses a combination of comparator, step & led machines to create an interactive bargraph display that changes with the turning of a pot (analog value). 

Source code: [led_fuel_gauge.ino](/tinkerspy/Automaton/blob/master/examples/led_fuel_gauge/led_fuel_gauge.ino)

#### Nuclear Missile_Launcher

Back in the days of the cold war both sides had intercontinental ballistic missiles aimed at each others cities buried in underground silo's that were manned by soldiers who could launch the missiles in the event of an surprise enemy strike. To avoid the disastrous consequences of a suicidal soldier launching a missile and starting off world war III on his own the launch controls were designed to require at least two different operators to trigger a launch.

In this example we use Automaton to build such a launch trigger mechanism on an Arduino Uno. Two buttons spaced widely apart that must be pressed within two seconds from each other to start a 10 second countdown (LED on pin 8). The countdown will trigger the missile ignition (pin 9).

We used two button machines connected to two bit machines with two timers to reset the bits after 2 second and a controller to check if both bit machines are in the on state. If that is the case the controller starts the countdown which fires the ignition to launch the missile.

Source code: [nuclear_missile_launcher.ino](/tinkerspy/Automaton/blob/master/examples/nuclear_missile_launcher/nuclear_missile_launcher.ino)

