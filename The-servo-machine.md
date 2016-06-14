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

Source code:
[Atm_servo.cpp](/tinkerspy/Automaton/blob/master/src/Atm_servo.cpp),
[Atm_servo.hpp](/tinkerspy/Automaton/blob/master/src/Atm_servo.hpp)

## Synopsis ##

```c++
```

### Atm_servo & begin( int pin1 ) ###

Attaches the servo machine to a digital pin.

### Atm_servo & step( int step_size, int step_time ) ###

Sets the step size and speed.

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

```c++
void setup() {
  ...
  servo.begin( 2 )
    .onFinish( led, led.EVT_BLINK );
  ...
}
```


### Atm_servo & position( int v ) ###


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
