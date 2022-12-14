Plays a single note or a pattern defined by an array of frequency/period/pause triplets through a passive piezo speaker or a standard speaker.

But it's more useful than that. By using the onNote() connector the player machine can be used to drive other machines or process any task at the intervals given in the pattern array. The onFinish() connector allows for chaining different patterns into 'songs'.

Note that the Arduino tone() command cannot control more than one pin at the same time so playing music on two player machines at the same time unfortunately won't work.

![Player](images/player.jpg)

<!-- md-tocify-begin -->
* [begin()](#atm_player--begin-int-pin---1-)  
* [play()](#atm_player--play-int-pat-int-patsize-)  
* [repeat()](#atm_player--repeat-int-v-)  
* [speed()](#atm_player--speed-int-v-)  
* [pitch()](#atm_player--pitch-int-v-)  
* [start()](#atm_player--start)  
* [stop()](#atm_player--stop)  
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
#include "musical_notes.h"

// Playback 'Frere Jacques' with button trigger and speed control 

Atm_player player;
Atm_button button;
Atm_analog speed;

int pattern[] = { 
  _G4, _N04, 0, _A4, _N04, 0, _B4, _N04, 0, _G4, _N04, 0, // Frere Jacques
  _G4, _N04, 0, _A4, _N04, 0, _B4, _N04, 0, _G4, _N04, 0, 
  _B4, _N04, 0, _C5, _N04, 0, _D5, _N04, _N04, // Dormez vous?
  _B4, _N04, 0, _C5, _N04, 0, _D5, _N04, _N04, 
  _D5, _N08, 0, _E5, _N08, 0, _D5, _N08, 0, _C5, _N08, 0, _B4, _N04, 0, _G4, _N04, 0, // Sonnez les matines 
  _D5, _N08, 0, _E5, _N08, 0, _D5, _N08, 0, _C5, _N08, 0, _B4, _N04, 0, _G4, _N04, 0, 
  _G4, _N04, 0, _D4, _N04, 0, _G4, _N04, _N04, // Ding dang dong
  _G4, _N04, 0, _D4, _N04, 0, _G4, _N04, _N04,
};

void setup() {
  player.begin( 19 ) // A passive buzzer or speaker on pin 19
    .play( pattern, sizeof( pattern ) )
    .repeat( -1 );

  button.begin( 2 ) // A button on pin 2 toggles playback on and off
    .onPress( player, player.EVT_TOGGLE );

  speed.begin( A0 ) // An analog pot on pin A0 controls playback speed
    .range( 50, 300 ) // From 50% to 300% of original speed
    .onChange( []( int idx, int v, int up ) {
      player.speed( v );    
    });
}

void loop() {
  automaton.run();
}
```
You'll also need the [musical_notes.h](/tinkerspy/Automaton/blob/master/examples/frere_jacques/musical_notes.h) file to compile this.

### Atm_player & begin( int pin = -1 ) ###

Initializes a player machine. If the pin argument is greater than -1 the machine will drive a piezo speaker connected to that pin through the Arduino tone() function. Leave out the pin argument if you want to drive something else through the onNote() connector.

```c++
void setup() {
 
  player.begin() 
    .play( pattern, sizeof( pattern ) )
    .onNote( true, led, led.EVT_ON )
    .onNote( false, led, led.EVT_OFF )
    .start();

}

```

In its default setting the player machine will play a single 880 hz tone for 50 milliseconds.

```c++

void setup() {
 
  player.begin( 19 ).start(); // Plays a 880 hz tone on pin 19
}

```

### Atm_player & play( int* pat, int patsize ) ###

Sets a pattern to be played by the player machine. The pattern should be an array of int with 3 entries for every tone.

Index | Function
---- | ----
0 | Frequency in herz
1 | Duration of note on in milliseconds
2 | Duration of note off in milliseconds

The patsize argument contains the total number of *bytes* in the pattern array.

The onFinish() connector can be used to chain different patterns together from a callback handler.

### Atm_player & play( uint32_t* pat, int patsize ) ###

This variant plays 32 bit patterns. This is probably not very useful for playing sounds over the standard tone() command but it enables the use of 32 bit patterns to control e.g. 32 output pins connected to 32 leds which is something many people like to do.

```c++
uint32_t pattern[] = {  // Bitmapped pattern (32 bits)
  B4INT( B00000000, B00000000, B00000001, B00100000 ), 100, 0,
  B4INT( B00000000, B00000000, B00000010, B00010000 ), 100, 0,
  B4INT( B00000000, B00000000, B00000001, B00001000 ), 100, 0,
  B4INT( B00000000, B00000000, B00000010, B00000100 ), 100, 0,
  B4INT( B00000000, B00000000, B00000001, B00000010 ), 100, 0,
  B4INT( B00000000, B00000000, B00000010, B00000001 ), 100, 0,
  B4INT( B00000000, B00000000, B00000001, B00000010 ), 100, 0,
  B4INT( B00000000, B00000000, B00000010, B00000100 ), 100, 0,
  B4INT( B00000000, B00000000, B00000001, B00001000 ), 100, 0,
  B4INT( B00000000, B00000000, B00000010, B00010000 ), 100, 0,
};

int pattern2[] = {  // Bitmapped pattern (16 bits)
  B2INT( B00000011, B11111111 ), 200, 0,
  B2INT( B00000000, B00000000 ), 200, 0,
};

void setup() {

  player.begin() 
    .play( pattern, sizeof( pattern ) );

```
The B4INT macro (defined in automaton.h) maps 4 bytes to a 32 bit unsigned integer so that we can use the handy Arduino Bxxxxxxxx constants. There's also a B2INT macro that does the same for 2 byte integers.

In 32 bit mode the lower 16 bits are in the *v* argument of the onNote() callback and the upper 16 bits are in the *up* argument.

### Atm_player & play( int freq, int period, int pause = 0 ) ###

Plays a single tone through the piezo speaker.

Argument | Function
---- | ----
freq | Frequency in herz
period | Duration of note on in milliseconds
pause | Duration of note off in milliseconds

```c++
void setup() {

  player.begin( 4 ) 
    .play( 440, 200 ); // Play a 440 hz tone for 200 ms

  button.begin( 2 )
    .onPress( player, player.EVT_START ); // When the button is pressed
}

```

### Atm_player & repeat( int v ) ###

Specifies how often the pattern should be repeated. Default is once (1). 
After the last repeat has finished the onFinish() connector is called.

### Atm_player & speed( int v ) ###

Modifies the speed of the pattern, value is a percentage.
At speed( 100 ) - the default - everything plays at the speed specified in the pattern array, at 50 everything plays at half speed, at 300 everything plays at triple speed, etc... 

The example below displays a pulsing led pattern on pins 4..9 defined by a bitmapped player pattern. The speed of the display can be controlled with a potmeter on pin A0 from 50% to 500% of the default pattern speed.

```c++
#include <Automaton.h>

Atm_player player; // A player machine
Atm_analog speed; // An analog machine for the potmeter

// These pins correspond to the bitmap bit positions
const uint8_t pin[] = { 4, 5, 6, 7, 8, 9 }; 

const int speedPotPin = A0;
const int speedMin = 50; // Half speed
const int speedMax = 500; // Quintuple speed

int pattern[] = {  // Bitmapped pattern
  B00000000, 100, 0, 
  B00001100, 100, 0, 
  B00011110, 100, 0, 
  B00111111, 100, 0, 
  B00011110, 100, 0, 
  B00001100, 100, 0, 
};

void setup() {
  player.begin() // No sound this time!
    .play( pattern, sizeof( pattern ) ) //  Set up the pattern
    .onNote( true, []( int idx, int v, int up ) { // Called on every note
      for ( int i = 0; i <= sizeof( pin ); i++ ) {
        pinMode( pin[i], OUTPUT ); // LED on/off according to bit  
        digitalWrite( pin[i], v & ( 1 << i ) ? HIGH : LOW ); 
      }    
    })
    .repeat( -1 ) // Repeat forever
    .start(); // Kickoff!
  speed.begin( speedPotPin ) 
    .range( speedMin, speedMax ) // Set the range for the pot values
    .onChange( []( int idx, int v, int up ) {
      player.speed( v ); // Set speed on every change of the potmeter
    });
}

void loop() {
  automaton.run();
}
```

Note that this time we don't use state machines to represent the leds. There's no need to since we're just switching them on and off and this way we need only 2 state machines instead of 10. State machines are a great tool, but there's no need to go overboard on them.

### Atm_player & pitch( int v ) ###

Modifies the pattern frequencies, value is a percentage.
At speed( 100 ) - the default - everything plays at the frequency specified in the pattern array, at 50 everything plays at half the frequency, at 300 everything plays at triple frequency, etc... 

```c++
#include <Automaton.h>

// Vary pitch and speed of a note by turning a pot

Atm_player player;
Atm_analog pot;

void setup() {
  player.begin( 19 )
    .play( 440, 100, 100 )
    .repeat( -1 )
    .start();
  pot.begin( A0 )
    .range( 30, 500 )
    .onChange( []( int idx, int v, int up ) {
       player.speed( v );
       player.pitch( v );
    });
}

void loop() {
  automaton.run();
}
```

### Atm_player & start() ###

Starts the player.

### Atm_player & stop() ###

Stops the player.

### Atm_player & onNote( {connector}, {connector-arg} ) ###

This is where it gets interesting. By using the onNote() connector the player machine can be used to drive other machines or process any task at the intervals given in the pattern array. The frequency value (which may very well contain something totally different) is passed on to a callback in the *v* argument.

```c++
void setup() {

  led.begin( 4 );
  
  player.begin()
    .play( pattern, sizeof( pattern ) )
    .onNote( true, led, led.EVT_ON ) // On note on
    .onNote( false, led, led.EVT_OFF ) // On note off
    .start();
}
```

The example below uses a callback combined with bitmapped values in the pattern/frequency field to trigger two solenoid valves in a particular order.

```c++

#include <Automaton.h>

Atm_player player;
Atm_button button;
Atm_led valve1, valve2;

int pattern[] = { 
  B00000001,  10, 0, // Frequency, Duration, Pause triplets
  B00000011, 980, 0,
  B00000001,  10, 0,
  B00000000,   0, 0,
};

// Open and close two solenoid valves in this order
// Valve 1: ___|^^^^^^^^|____
// Valve 2: ____|^^^^^^|_____

void setup() {
  valve1.begin( 4 );
  valve2.begin( 5 );
  player.begin() 
    .play( pattern, sizeof( pattern ) )
    .onNote( true, []( int idx, int v, int up ) {
      valve1.trigger( ( v & B00000001 ) > 0 ? valve1.EVT_ON : valve1.EVT_OFF );
      valve2.trigger( ( v & B00000010 ) > 0 ? valve2.EVT_ON : valve2.EVT_OFF );	  
    });
  button.begin( 2 )
    .onPress( player, player.EVT_START );
}

void loop() {
  automaton.run();
}
```

You can easily control 8 or more valves this way. 

### Atm_player & onFinish( {connector}, {connector-arg} ) ###

A connector that is called when playing stops (after the last repeat of a pattern has finished).

The code below uses onFinish() to chain two different patterns together.

```c++
#include <Automaton.h>

Atm_player player;
Atm_button button;

int pattern1[] = { // Odd pattern
  440, 100, 0, 
  587, 100, 0,
  880, 100, 100,
};

int pattern2[] = { // Even pattern
  880, 100, 0, 
  587, 100, 0,
  440, 100, 100,
};

void callback( int idx, int v, int up ) {
  static int cnt = 0;
  if ( cnt % 2 == 0 ) {
    player.play( pattern2, sizeof( pattern2 ) ); // Even counts
  } else {
    player.play( pattern1, sizeof( pattern1 ) ).trigger( player.EVT_STOP ); // Odd counts
  }
  cnt++;  
}

void setup() {

  player.begin( 4 ) 
    .play( pattern1, sizeof( pattern1 ) )
    .onFinish( callback );

  button.begin( 2 )
    .onPress( player, player.EVT_TOGGLE );
}

void loop() {
  automaton.run();
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
