A state machine that handles commands coming in over a serial line (Stream), parses and interprets them and fires off a handler callback. Make sure you set the Arduino monitor to send newlines when testing.

* [begin()](#atm_command--begin-stream--stream-char-buffer-int-size-)
* [onCommand()](#atm_command--oncommandvoid-callback-int-idx--const-char--cmds--)
* [separator()](#atm_command--separator-const-char-sep-)
* [lookup()](#int-lookup-int-id-const-char--cmdlist-)
* [arg()](#char--arg-int-id-)
* [onSwitch()](#machine--onswitch-swcb_sym_t-callback-)

## Synopsis ##

```c++
#include <Automaton.h>
#include <Atm_command.h>

char cmd_buffer[80];
Atm_command cmd;
Factory factory;

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
  Serial.begin( 9600 );
  cmd.begin( &Serial, cmd_buffer, sizeof( cmd_buffer ) );
  cmd.onCommand( cmd_callback, cmdlist );
  factory.add( cmd );
}

void loop() {
  factory.cycle();
}
```

### Atm_command & begin( Stream * stream, char buffer[], int size ) ###

Connects a stream and a command line character buffer to the state machine.

```c++
char cmd_buffer[80];
Atm_command cmd;

void setup() {
  cmd.begin( &Serial, cmd_buffer, sizeof( cmd_buffer ) );
  ...
}
```

### Atm_command & onCommand(void (*callback)( int idx ), const char * cmds  ) ###

Sets a callback handler and sets the list of commands the state machine should look out for.

```c++
const char cmdlist[] = "pulse lock";

void setup {
  ...
  cmd.onCommand( cmd_callback, cmdlist );
}
```

The *idx* argument to the callback contains the position of the command (arg 0) in the cmdlist string. If in the example above the detected command is 'pulse' (case insensitive), idx will be 0, if it is 'lock' idx will be 1. If no match is found idx will be -1 and the unmatched command will be available with cmd.arg( 0 ).


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

This command looks up the second token in the buffer to see if it contains either *ON* or *OFF* (case insensitive match). Returns 0 for the first and 1 for the second. Returns -1 if the argument was not found. To be used inside the callback routine that's called after an incoming command line has been parsed.

### char * arg( int id ) ###

Returns the n-th argument found in the cmdline string. To be used inside the callback routine that's called after an incoming command line has been parsed.

### Machine & onSwitch( swcb_sym_t callback ) ###

To monitor the behavior of this machine you may connect a monitoring function with the Machine::onSwitch() method. 

```c++
cmd.onSwitch( atm_serial_debug::onSwitch );
```
