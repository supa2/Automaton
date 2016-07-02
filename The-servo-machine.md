The servo state machine controls a rotary servo.


<!-- md-tocify-begin -->
* [begin()]()  
* [onChange()](#atm_servo--onchange-connector-connector-arg-)  
* [onFinish()](#atm_servo--onfinish-connector-connector-arg-)  
* [step()](#atm_servo--position-int-v-)  
* [position()](#atm_servo--position-int-v-)  
* [state()](#int-state-void-)  
* [trace()](#atm_servo--trace-stream--stream-)  

<!-- md-tocify-end -->

The Atm_servo code has been moved to a separate library:  
https://github.com/tinkerspy/Automaton-Servo


## Synopsis ##

```c++
#include <Automaton.h>
#include <Atm_servo.h>

// Sweep the servo slowly from 0 to 180 degrees on two buttons
// Blink a led when position is reached

Atm_servo servo;
Atm_button button1, button2;
Atm_led led;

void setup() {
  led.begin( 4 )
    .blink( 500, 0 , 1 ); // Led indicates movement finished 

  servo.begin( 5 ) // Servo on pin 5
    .step( 1, 20 ) // 1 degree steps in 20 ms (slow)
    .onFinish( led, led.EVT_BLINK ); // Blink a led when position reached

  button1.begin( 2 )
    .onPress( servo, 0 ); // Button on pin 2 turns servo to pos 0

  button2.begin( 3 )
    .onPress( servo, 180 ); // Button on pin 3 turns servo to pos 180
}

void loop () {
  automaton.run();
}
```

### Atm_servo & begin( int pin1 ) ###

Attaches the servo machine to a digital pin.

### Atm_servo & step( int step_size, int step_time ) ###

Sets the step size and speed. The step_size parameter sets the numer of degrees the servo should move at a time. 
The step_time parameter sets the number of milliseconds reserved for the servo to move step_size degrees.

Default setting is step( 180, 0), the servo will move immediately in zero time. This will work fine if you want your servo to move at full speed and you don't care when it finishes. In case of a continuous rotation servo, leave step() at its default setting.

```c++
void setup() {
  servo.begin( 5 )
    .step( 1, 10 ); // Move 1 degree in 10 ms
  servo.begin( 5 )
    .step( 5, 20 ); // Move 5 degrees in 20 ms
  servo.position( 180 ); // Start moving!
}
```

### Atm_servo & onChange( {connector}, {connector-arg} ) ###

Specifies a machine or callback to be triggered whenever the servo machine changes position.

```c++
void setup() {
  ...
  servo.begin( 2 )
    .onChange( led, led.EVT_BLINK );
  ...
}
```


### Atm_servo & onChange( bool status, {connector}, {connector-arg} ) ###

Specifies a machine or callback to be triggered whenever the servo machine changes position in the clockwise (status true or ATM_UP) or counterclockwise (status false or ATM_DOWN) direction.

```c++
void setup() {
  ...
  servo.begin( 2 )
    .onChange( ATM_UP, upLed, upLed.EVT_BLINK )
    .onChange( ATM_DOWN, downLed, downLed.EVT_BLINK );
  ...
}
```

### Atm_servo & onFinish( {connector}, {connector-arg} ) ###

Specifies a machine or callback to be triggered whenever the servo machine reaches its destination position.
This is only useful when a step size and time are specified, otherwise the servo will report a finish event immediately
after a new position has been set.

```c++
void setup() {
  ...
  servo.begin( 2 )
    .step( 1, 10 )
    .onFinish( led, led.EVT_BLINK );
  servo.position( 180 );
  ...
}
```


### Atm_servo & position( int v ) ###

Set the servo position from 0 to 180 degrees.
If the new position differs from the current posituion, the servo will start moving at the speed of step_size degrees per step_time milliseconds until it reaches the new position.

```c++
void setup() {
  servo.begin( 2 ) 
    .position( 180 );
}
```

### int state( void ) ###

Returns the integer value of the current servo position (at least the machines view of it).

```c++
```

### Atm_servo & trace( Stream & stream ) ###

To monitor the behavior of this machine you may log state change events to a Stream object like Serial.

```c++
Serial.begin( 9600 );
servo.trace( Serial );
```
