Not just a music player but a programmable pattern generator.

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

### Atm_player& play( int* pat, int patsize ) ###

Pattern chaining requires extra repeat() in callback.

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
    player.play( pattern2, sizeof( pattern2 ) ).repeat( 1 );
  } else {
    player.play( pattern1, sizeof( pattern1 ) );
  }
  cnt++;  
}

void setup() {
  Serial.begin( 9600 );
//  player.trace( Serial );

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

### Atm_player& repeat( int v ) ###

### Atm_player& speed( int v ) ###

### Atm_player& onNote( {connector}, {connector-arg} ) ###

### Atm_player& onFinish( {connector}, {connector-arg} ) ###

### trace( Stream & stream ) ###