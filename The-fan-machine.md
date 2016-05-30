The Atm_fan machine has a single purpose. It converts a single event trigger to up to 4 other triggers.

![Fade](images/fan-small.jpg)

<!-- md-tocify-begin -->
* [begin()](#atm_player--begin-)  
* [onInput()](#atm_player--onnote-connector-connector-arg-)  
* [trace()](#atm_player--trace-stream--stream-)  
* [EVT_INPUT](#evt_input)  

<!-- md-tocify-end -->

## Synopsis ##

```c++
#include <Automaton.h>

Atm_fan fan;
Atm_led led[4];
Atm_button button;
Appliance app;

void setup() {
 
  app.component( led[0]. begin( 4 ) );
  app.component( led[1]. begin( 5 ) );
  app.component( led[2]. begin( 6 ) );
  app.component( led[3]. begin( 7 ) );

  app.component(
    fan.begin() 
      .onInput( led[0], led[0].EVT_TOGGLE )
      .onInput( led[1], led[1].EVT_TOGGLE )
      .onInput( led[2], led[2].EVT_TOGGLE )
      .onInput( led[3], led[3].EVT_TOGGLE )
  );

  app.component( // One button toggles 4 leds
    button.begin( 2 )
      .onPress( fan, fan.EVT_TOGGLE )
  );

}

void loop() {
  app.run();
}
```

### Atm_player & begin( void ) ###

Initializes a fan machine. 

### Atm_player & onInput( {connector}, {connector-arg} ) ###

Connects one of the four fan outputs to a machine or callback.

```c++
void setup() {
  ...
  app.component( 
    fan.begin()
      .onInput( led1, led1.EVT_ON ) 
      .onInput( led2, led2.EVT_ON ) 
      .onInput( led3, led3.EVT_ON ) 
      .onInput( led4, led4.EVT_ON ) 
  );
  ...
}
```

### Atm_fan & trace( Stream & stream ) ###

To monitor the behavior of this machine you may log state change events to a Stream object like Serial.

```c++
Serial.begin( 9600 );
fan.trace( Serial );
```

### EVT_INPUT ###

The fan machine triggers all its outputs on receipt of this event.

```c++
  fan.trigger( fan.EVT_INPUT );
```

