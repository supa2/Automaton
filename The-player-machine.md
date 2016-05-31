Plays a single note or a pattern defined by an array of frequency/period/pause triplets through a piezo speaker.

But it's more useful than that. By using the onNote() connector the player machine can be used to drive other machines or process any task at the intervals given in the pattern array. The onFinish() connector allows for chaining different patterns into 'songs'.

![Player](images/player.jpg)

<!-- md-tocify-begin -->
* [begin()](#atm_player--begin-int-pin---1-)  
* [play()](#atm_player--play-int-pat-int-patsize-)  
* [repeat()](#atm_player--repeat-int-v-)  
* [speed()](#atm_player--speed-int-v-)  
* [onNote()](#atm_player--onnote-connector-connector-arg-)  
* [onFinish()](#atm_player--onfinish-connector-connector-arg-)  
* [trace()](#atm_player--trace-stream--stream-)  
* [EVT_START](#evt_start)  
* [EVT_STOP](#evt_stop)  
* [EVT_TOGGLE](#evt_toggle)  

<!-- md-tocify-end -->

## Synopsis ##

```c++
#include <Automaton.h>

Atm_player player;
Atm_button button;
Appliance app;

int pattern[] = { 
  440, 100, 0, // Frequency, Duration, Pause triplets
  587, 100, 0,
  880, 100, 100,
};

void setup() {
 
  app.component( 
    player.begin( 4 ) // Piezo speaker on pin 4
      .play( pattern, sizeof( pattern ) )
  );

  app.component( // Button on pin 2
    button.begin( 2 )
      .onPress( player, player.EVT_START ) // Linked by an event
  );

}

void loop() {
  app.run();
}
```

### Atm_player & begin( int pin = -1 ) ###

Initializes a player machine. If the pin argument is greater than -1 the machine will drive a piezo speaker connected to that pin through the Arduino tone() function. Leave out the pin argument if you only want to use the onNote() connector.

```c++
void setup() {
 
  app.component( 
    player.begin() 
      .play( pattern, sizeof( pattern ) )
      .onNote( true, led, led.EVT_ON )
      .onNote( false, led, led.EVT_OFF )
      .trigger( player.EVT_START )
  );

}


```

### Atm_player & play( int* pat, int patsize ) ###

Sets a pattern to be played by the player machine. The pattern should be an array of int with 3 entries for every tone.

Index | Function
---- | ----
0 | Frequency in herz
1 | Duration of note on
2 | Duration of note off

The patsize argument contains the total number of *bytes* in the pattern array.

The onFinish() connector can be used to chain different patterns together from a callback handler.

### Atm_player & play( int freq, int period, int pause = 0 ) ###

Plays a single tone through the piezo speaker.

Argument | Function
---- | ----
freq | Frequency in herz
period | Duration of note on
pause | Duration of note off

```c++
void setup() {

  app.component( 
    player.begin( 4 ) 
      .play( 440, 200 ) // Play a 440 hz tone for 200 ms
  );

  app.component(
    button.begin( 2 )
      .onPress( player, player.EVT_START ) // When the button is pressed
  );
}

```

### Atm_player & repeat( int v ) ###

Specifies how often the pattern should be repeated. Default is once (1). 
After the last repeat has finished the onFinish() connector is called.

### Atm_player & speed( int v ) ###

Modifies the speed of the pattern, value is a percentage.
At speed( 100 ) - the default - everything plays at the speed specified in the pattern array, at 50 everything plays at half speed, at 300 everything plays at triple speed, etc... 

The example below displays a pulsing led pattern on pins 4..9 defined by a bitmapped pattern. The speed of the display can be controlled with a potmeter on pin A0.

```c++
#include <Automaton.h>

Atm_player player;
Atm_analog speed;
Appliance app;

const int startLedPin = 4;
const int speedPotPin = A0;
const int speedMin = 50;
const int speedMax = 500;

int pattern[] = { 
  B00000000, 100, 0, 
  B00001100, 100, 0, 
  B00011110, 100, 0, 
  B00111111, 100, 0, 
  B00011110, 100, 0, 
  B00001100, 100, 0, 
};

void setup() {
  app.component( 
    player.begin( -1 )
      .play( pattern, sizeof( pattern ) )
      .onNote( true, []( int idx, int v, int up ) { // Called on every note
        for ( int i = 0; i < 6; i++ ) {
          pinMode( i + startLedPin, OUTPUT ); // LED on/off according to bit  
          digitalWrite( i + startLedPin, ( v & ( 1 << i ) ) ? HIGH : LOW ); 
        }    
      })
      .repeat( ATM_COUNTER_OFF ) // This means forever
      .trigger( player.EVT_START )
  );
  app.component( 
    speed.begin( speedPotPin )
      .range( speedMin, speedMax )
      .onChange( []( int idx, int v, int up ) {
        player.speed( v ); // Set speed on every change of the potmeter
      })
  );
}

void loop() {
  app.run();
}
```

Note that we don't use state machines to represent the leds. There's no need to since we're just switching them on and off and this way we need only 2 state machines instead of 10. State machines are a great tool, but there's no need to go overboard on them.

### Atm_player & onNote( {connector}, {connector-arg} ) ###

This is where it gets interesting. By using the onNote() connector the player machine can be used to drive other machines or process any task at the intervals given in the pattern array. The frequency value (which may very well contain something totally different) is passed on to a callback in the *v* argument.

```c++
void setup() {

  app.component( led.begin( 4 ) );
  
  app.component( 
    player.begin()
      .play( pattern, sizeof( pattern ) )
      .onNote( true, led, led.EVT_ON ) // On note on
      .onNote( false, led, led.EVT_OFF ) // On note off
  );

}
```

The example below uses a callback combined with 'magic' values in the pattern/frequency field to trigger two solenoid valves in a particular order.

```c++
#include <Automaton.h>

Atm_player player;
Atm_button button;
Atm_led valve1, valve2;
Appliance app;

int pattern[] = { 
  1,  10, 0, // Frequency, Duration, Pause triplets
  2, 980, 0,
  3,  10, 0,
  4,   0, 0,
};

// Open and close two solenoid valves in this order
// Valve 1: ___|^^^^^^^^|____
// Valve 2: ____|^^^^^^|_____

void callback( int idx, int v, int up ) {
  if ( up ) {
    switch ( v ) {
      case 1:
        valve1.trigger( valve1.EVT_ON );
        return;
      case 2:
        valve2.trigger( valve2.EVT_ON );
        return;
      case 3:
        valve2.trigger( valve2.EVT_OFF );
        return;
      case 4:
        valve1.trigger( valve1.EVT_OFF );
        return;
    } 
  }
}

void setup() {

  app.component( 
    player.begin() 
      .play( pattern, sizeof( pattern ) )
      .onNote( callback )
  );

  app.component(
    button.begin( 2 )
      .onPress( player, player.EVT_START )
  );

}

```

### Atm_player & onFinish( {connector}, {connector-arg} ) ###

A connector that is called when playing stops (after the last repeat of a pattern has finished).

The code below uses onFinish() to chain two different patterns together.

```c++
#include <Automaton.h>

Atm_player player;
Atm_button button;
Appliance app;

int pattern1[] = { 
  440, 100, 0, 
  587, 100, 0,
  880, 100, 100,
};

int pattern2[] = { 
  880, 100, 0, 
  587, 100, 0,
  440, 100, 100,
};

void callback( int idx, int v, int up ) {
  static int cnt = 0;
  if ( cnt % 2 == 0 ) {
    player.play( pattern2, sizeof( pattern2 ) );
  } else {
    player.play( pattern1, sizeof( pattern1 ) ).trigger( player.EVT_STOP );
  }
  cnt++;  
}

void setup() {

  app.component( 
    player.begin( 4 ) 
      .play( pattern1, sizeof( pattern1 ) )
      .onFinish( callback )
  );

  app.component(
    button.begin( 2 )
      .onPress( player, player.EVT_TOGGLE )
  );
}

void loop() {
  app.run();
}
```


### Atm_player & trace( Stream & stream ) ###

To monitor the behavior of this machine you may log state change events to a Stream object like Serial.

```c++
Serial.begin( 9600 );
player.trace( Serial );
```

### EVT_START ###

The player machine starts playback on receipt of this event.

```c++
  player.trigger( player.EVT_START );
```

### EVT_STOP ###

The player machine stops playback on receipt of this event.

```c++
  player.trigger( player.EVT_STOP );
```

### EVT_TOGGLE ###

The player machine toggle playback on or off on receipt of this event.

```c++
  player.trigger( player.EVT_TOGGLE );
```
