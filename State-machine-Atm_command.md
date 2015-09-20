A state machine that handles commands coming in over a serial line (Stream), parses and interprets them and fires off a handler callback.

* begin()
* onCommand()
* separator()
* lookup()
* arg()
* onSwitch()

### Atm_command & begin( Stream * stream, char buffer[], int size ) ###

Connects a stream and a command line character buffer to the state machine.

```c++
#include <Automaton.h>
#include <Atm_command.h>

char cmd_buffer[80];
Atm_command cmd;

enum { PULSE, LOCK };
const char cmdlist[] = "pulse lock";
enum { ON, OFF };
const char on_off[] = "on off";

void cmd_callback( int idx ) {

  switch ( idx ) {
    case PULSE :
      // Do something for PULSE
      return;
    case LOCK :
      switch ( cmd.lookup( 1, on_off) ) {
        case ON : 
          // Do something for LOCK ON
          return;
        case OFF : 
          // Do something for LOCK OFF
          return;
      }
      return;
  }
}

void setup() {
  cmd.begin( &Serial, cmd_buffer, sizeof( cmd_buffer ) );
  cmd.onCommand( cmd_callback, cmdlist );
}
```

### Atm_command & onCommand(void (*callback)( int idx ), const char * commands  ) ###

Sets a callback handler and sets the list of commands the state machine should look out for.

```c++
const char cmdlist[] = "pulse lock";

void setup {
  ...
  cmd.onCommand( cmd_callback, cmdlist );
}
```

### Atm_command & separator( const char sep[] ) ###

Sets the separator list used to parse the incoming command line. The separator characters are used divide the command line in arguments. Default is ' '. Multiple separator characters are merged into one.

```c++
void setup {
  ...
  cmd.separator( ",; " );
}
```

### int lookup( int id, const char * cmdlist ) ###

Look up the argument defined by *id* in the command list.

```c++
  int idx = cmd.lookup( 1, "on off" )
```

This command looks up the second token in the buffer to see if it contains either *ON* or *OFF* (case insensitive match). Returns 0 for the first and 1 for the second. Returns -1 if the argument was not found.
