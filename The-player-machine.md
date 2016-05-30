Plays a single note or an array of frequency/period/pause triplets through a piezo speaker.

But it's more useful than that. By using the onNote() connector the player machine can be used to drive other machines or process any task at the intervals given in the pattern array.

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
    player.begin( 3 ) 
      .play( pattern, sizeof( pattern ) )
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

### Atm_player& begin( int pin = -1 ) ###

Initializes a player machine. If the pin argument is greater than -1 the machine will drive a piezo speaker connected to that pin through the Arduino tone() function. Leave the pin argument at -1 if you only want to use the onNote() connector.

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

### Atm_player& play( int* pat, int patsize ) ###

Sets a pattern to be played by the player machine. The pattern should be an array of int with 3 entries for every tone.

Index | Function
---- | ----
0 | Frequency in herz
1 | Duration of note on
2 | Duration of note off

The patsize argument contains the total number of *bytes* in the pattern array.

The onFinish() connector can be used to chain different patterns together from a callback handler.

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
    player.begin( 3 ) 
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

### Atm_player& play( int freq, int period, int pause = 0 ) ###

Plays just one tone through the piezo speaker.

Argument | Function
---- | ----
freq | Frequency in herz
period | Duration of note on
pause | Duration of note off

```c++
void setup() {

  app.component( 
    player.begin( 3 ) 
      .play( 440, 200 ) // Play a 440 hz tone for 200 ms
  );

  app.component(
    button.begin( 2 )
      .onPress( player, player.EVT_START )
  );
}

```

### Atm_player& repeat( int v ) ###

Specifies how often the pattern should be repeated. Default is once (1). 
After the last repeat has finished the onFinish() connector is called.

### Atm_player& speed( int v ) ###

Modifies the speed of the pattern, value is a percentage.
At speed( 100 ) - the default - everything plays at the speed specified in the pattern array, at 50 everything plays at half speed, at 300 everything plays at triple speed, etc... 

```c++
void setup() {

  app.component( 
    player.begin( 3 ) 
      .play( pattern, sizeof( pattern ) )
      .repeat( ATM_COUNTER_OFF) // Repeat forever!
  );

  app.component( 
    potmeter.begin( A0 ) // Vary the playback speed with a pot on A0
      .range( 20, 300 )
      .onChange( []( int idx, int v, int up ) {
         player.speed( v );
       });
  );

}

```

### Atm_player& onNote( {connector}, {connector-arg} ) ###

This is where it gets interesting. By using the onNote() connector the player machine can be used to drive other machines or process any task at the intervals given in the pattern array. The frequency value (which may very well contain something totally different) is passed on to a callback in the *v* argument.

```c++
void setup() {

  app.component( led.begin( 4 ) );
  
  app.component( 
    player.begin()
      .onNote( true, led, led.EVT_ON ) // On note on
      .onNote( false, led, led.EVT_OFF ) // On note off
  );

}
```

### Atm_player& onFinish( {connector}, {connector-arg} ) ###

A connector that is called when playing stops (after the last repeat of a pattern has finished).

```c++
void setup() {

  app.component( 
    player.begin( 3 ) 
      .play( pattern1, sizeof( pattern1 ) )
      .onFinish( callback )
  );

}

```

```c++

int pattern[] = { 
  1,  10, 0, // Frequency, Duration, Pause triplets
  2, 980, 0,
  3,  10, 0,
  4,  10, 0,
};

// Switch two solenoids in order
// Solenoid 1: ___|^^^^^^^^|____
// Solenoid 2: ____|^^^^^^|_____

void callback( int idx, int v, int up ) {
  switch ( v ) {
    case 1:
      solenoid1.trigger( solenoid1.EVT_ON );
      return;
    case 2:
      solenoid2.trigger( solenoid2.EVT_ON );
      return;
    case 3:
      solenoid2.trigger( solenoid2.EVT_OFF );
      return;
    case 4:
      solenoid1.trigger( solenoid1.EVT_OFF );
      return;
  } 
}

void setup() {

  app.component( 
    player.begin() 
      .play( pattern1, sizeof( pattern1 ) )
      .onNote( callback )
  );

}

```


### Atm_player & trace( Stream & stream ) ###

To monitor the behavior of this machine you may log state change events to a Stream object like Serial.

```c++
Serial.begin( 9600 );
player.trace( Serial );
```
