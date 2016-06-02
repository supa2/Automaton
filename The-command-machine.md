A state machine that handles commands coming in over a serial line (Stream), parses and interprets them and fires off a handler callback. Make sure you set the Arduino monitor to 9600 baud and to send newlines when testing. Atm_command handles any combination of \r and \n as a line ending and matches whole words in a case insensitive manner.

![Command](images/command-small.png)

<!-- md-tocify-begin -->
* [begin()](#atm_command--begin-stream--stream-char-buffer-int-size-)  
* [onCommand()](#atm_command--oncommand-connector-connector-arg-)  
* [list()](#atm_command--list-const-char--cmds-)  
* [separator()](#atm_command--separator-const-char-sep-)  
* [lookup()](#int-lookup-int-id-const-char--cmdlist-)  
* [arg()](#char--arg-int-id-)  
* [trace()](#atm_command--trace-stream--stream-)  

<!-- md-tocify-end -->

## Synopsis ##

```c++
#include <Automaton.h>

char cmd_buffer[80];
Atm_command cmd;
Appliance app;

enum { CMD_HIGH, CMD_LOW, CMD_READ, CMD_AREAD, CMD_AWRITE, CMD_MODE_INPUT, CMD_MODE_OUTPUT, CMD_MODE_PULLUP };
const char cmdlist[] = "high low read aread awrite mode_input mode_output mode_pullup";

void cmd_callback( int idx, int v, int up ) {
  int pin = atoi( cmd.arg( 1 ) );
  switch ( v ) {
    case CMD_HIGH: 
      digitalWrite( pin, HIGH );
      return;
    case CMD_LOW:
      digitalWrite( pin, LOW );
      return;
    case CMD_READ:
      Serial.println( digitalRead( pin ) );
      return;
    case CMD_AREAD:
      Serial.println( analogRead( pin ) );
      return;
    case CMD_AWRITE:
      analogWrite( pin, atoi( cmd.arg( 2 ) ) );
      return;
    case CMD_MODE_INPUT:
      pinMode( pin, INPUT );
      return;
    case CMD_MODE_OUTPUT:
      pinMode( pin, OUTPUT );
      return;
    case CMD_MODE_PULLUP:
      pinMode( pin, INPUT_PULLUP );
      return;
  }
}

void setup() {
  Serial.begin( 9600 );
  app.component( 
    cmd.begin( Serial, cmd_buffer, sizeof( cmd_buffer ) )
      .list( cmdlist )
      .onCommand( cmd_callback )
  );

}

void loop() {
  app.run();
}
```

This simple example turns your Arduino into a serial device to read and manipulate pins. 

```
// Set pin 4 to output and turn it on
mode_output 4 
high 4

// Set pin 5 to input_pullup and read its state
mode_pullup 5
read 5
1
```

### Atm_command & begin( Stream & stream, char buffer[], int size ) ###

Connects a stream and a command line character buffer to the state machine.

```c++
char cmd_buffer[80];
Atm_command cmd;
Appliance app;

void setup() {
  app.component( cmd.begin( Serial, cmd_buffer, sizeof( cmd_buffer ) );
  ...
}
```

### Atm_command & onCommand( {connector}, {connector-arg} ) ###

Sets a callback handler.

```c++
const char cmdlist[] = "pulse lock";

void setup {
  ...
  cmd.list( cmdlist );
  cmd.onCommand( cmd_callback );
}
```

The *id_cmd* argument to the callback contains the position of the command (arg 0) in the cmdlist string. If in the example above the detected command is 'pulse' (case insensitive), idx will be 0, if it is 'lock' id_cmd will be 1. If no match is found id_cmd will be -1 and the unmatched command will be available with cmd.arg( 0 ).

### Atm_command & list( const char * cmds ) ###

sets the list of commands the state machine should look out for.

```c++
const char cmdlist[] = "pulse lock";

void setup {
  ...
  cmd.list( cmdlist );
  cmd.onCommand( cmd_callback );
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

This command looks up the second token in the buffer to see if it contains either *ON* or *OFF* (case insensitive match). Returns 0 for the first and 1 for the second. Returns -1 if the argument was not found. To be used inside the callback routine that's called after an incoming command line has been parsed.

### char * arg( int id ) ###

Returns the n-th argument found in the cmdline string. To be used inside the callback routine that's called after an incoming command line has been parsed.

### Atm_command & trace( Stream & stream ) ###

To monitor the behavior of this machine you may log state change events to a Stream object like Serial.

```c++
Serial.begin( 9600 );
cmd.trace( Serial );
```
