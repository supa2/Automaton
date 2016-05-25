*Machine* is an abstract class, which means it cannot be instantiated directly. It must be subclassed first. These subclasses define *Finite State Machines* which inherit all *Machine* functionality and *can* be instantiated (used as an object). Every Machine subclass should define its own version of the *begin*(), *event*() and *action*() methods. 

<!-- md-tocify-begin -->
* [begin()](#machine--begin-const-state_t-tbl-state_t-width-)  
* [event()](#int-event-int-id-)  
* [action()](#void-action-int-id-)  
* [state()](#machine--state-void-)  
* [trigger()](#int-trigger-int-event-)  
* [sleep()](#uint8_t-sleep-int8_t-v-)  
* [cycle()](#machine--cycle-uint32_t-time-)  
* [trace()](#machine--trace-stream--stream-)  
* [Machine variables](#machine-variables)  

<!-- md-tocify-end -->

## Initialization ##

### Machine & begin( const state_t* tbl, state_t width ) ###

Calls the Machine base class initialization code. Links a *state transition table* to the machine. Each machine subclass should define its own begin() method which, in turn should call Machine::begin() to get the ball rolling.

```c++
Machine::begin( state_table, ELSE );
```

The *ELSE* is an identifier from the *STATES* enum section of the class definition. It is used to determine the dimensions of the data structure.

For this reason each *STATES* enum must end with an *ELSE* event.

```c++
const static state_t state_table[] PROGMEM = {
/*                  ON_ENTER    ON_LOOP    ON_EXIT  EVT_INPUT   EVT_EOL   ELSE */
/* IDLE     */            -1,        -1,        -1,  READCHAR,       -1,    -1,
/* READCHAR */  ENT_READCHAR,        -1,        -1,  READCHAR,     SEND,    -1,
/* SEND     */      ENT_SEND,        -1,        -1,        -1,       -1,  IDLE,
};
Machine::begin( state_table, ELSE );
```

The *ELSE* event is automatic (generates no call to the event() method).

### int event( int id ) ###

Event handler. This is a *pure virtual method* (which means every subclass of Machine must implement it). This handler will be called for every event in the current state where the corresponding table entry is not -1. The id argument contains the event id for which the handler is called.

The Machine class calls the subclass event() method to find out if a certain event (id) has occured. The event() method should return 1 if the requested event has occured and 0 if it hasn't.  

The state machine makes its state change decisions on the basis of what this handler returns combined with the contents of the state table.

```c++
int Atm_command::event( int id )
{
  switch ( id ) {
    case EVT_INPUT :
      return _stream->available();
    case EVT_EOL :
      return _buffer[_bufptr-1] == _eol || _bufptr >= _bufsize;
  }
  return 0;
}
```

Failing to implement the event() method in a subclass generates the following compiler error: 

```
Cannot declare variable <objectname> to be of abstract type...
```

### void action( int id ) ###

Action handler. This is a pure virtual method (which means every subclass of Machine must implement it). This handler will be called for every action in the current state where the corresponding table entry is not -1*. The id argument contains the action id for which the handler is called.

```c++
void Atm_led::action( int id )
{
  switch ( id ) {
    case ENT_INIT :
      counter.set( repeat_count );
      digitalWrite( pin, LOW );
      return;
    case ENT_ON :
      decrement( counter );
      digitalWrite( pin, HIGH );
      return;
    case ENT_OFF :
      digitalWrite( pin, LOW );
      return;
  }
}
```

Just before every state change the state machine calls the action handler with the *ATM_ON_SWITCH* id. In that case the 'next' variable contains the new state and the 'current' variable contains the old (present) state.

Failing to implement the action() method in a subclass generates the following compiler error: 

```
Cannot declare variable <objectname> to be of abstract type...
```

* NOTE: The action method is actually called for every ON_ENTER event of every state regardless of what's in the ON_ENTER column. Any signed 8bit integer value in that column is passed on in 'id'. The same goes for ON_ENTER. In the case of ON_LOOP the -1 value does not trigger a call to action().

## States ##

### Machine & state( void ) ###

Returns the current (numeric) state of the machine.

```c++
if ( led.state() != led.IDLE ) {
	led.trigger( led.EVT_OFF );
}
```

Versions before 0.2.0 supported setting a machine's state with this method. This is now strongly advised against and the state( new_state ) method has been made protected (only accessible from inside a machine), if you insist on allowing this for your own custom machines create a custom method that calls it. The preferred way of changing states is through the trigger() methods.

By default state() will return the current machine state (the row of the state table that is currently active) but a machine's author may override the state() method to make it return whatever state makes sense in the context of the machine. For example the Atm_encoder's state() method returns the value of the internal counter instead.

### int trigger( int event ) ###

Triggers an event for the current state. If there's a positive number in the event column for the current state the machine will switch to that state on the next cycle. The method will return 1 if the trigger has resulted in a state change or 0 if it hasn't.

A machine's author may subclass the trigger() method to handle more (or less) than the machine's internal events thereby controlling the machine's (incoming) interface to the world.

```c++
void setup() {
  led1.begin( 4 );
  app.component( led1 );
  led1.trigger( led1.EVT_BLINK );
}
```

The trigger method will cycle a machine up to 8 times until it has become responsive (which means that there is an non negative value in the corresponding state table column for the current state. Then the machine will be cycled twice, once for picking up the event, once for the ensuing state change.

```c++
  led1.begin( 4 );
  led1.trigger( led1.EVT_BLINK );
```

Triggered events can not be stored but are processed immediately or are discarded.

## Scheduling ##
----------

### uint8_t sleep( [int8_t v] ) ###

Returns true if the object is in sleeping state (which is the case if the current state has the ATM\_SLEEP constant on the ON\_LOOP column). A machine in a sleeping state does not execute its event loop and does not call its action() handler, it does, however, process incoming messages.

By setting the v argument to a non zero value the machine is brought into a sleeping state. By setting the v value to zero the machine is put to sleep.

```c++
if ( led1.sleep() ) {
  ...
}

led1.sleep( 1 );
```


### Machine & cycle( [uint32_t time] ) ###

Executes one cycle of the state machine. Normally only called by the appliance class but can also be used directly inside the Arduino loop() function to bypass the appliance class altogether.

```c++
void loop()
{
    led1.cycle();
    led2.cycle();
    led3.cycle();
}
```

When the time argument is specified and greater than zero, the cycle() method will cycle until the corresponding number of milliseconds has passed. This can be handy if you want to run state machines in a sequential pattern (usually from the setup method) like this:

```c++
void setup()
{
  // Blink a led slowly three times
  led.begin( 4 ).blink( 500, 500, 3 ).trigger( Atm_led::EVT_BLINK );

  // Wait until it finishes
  while ( led.cycle().state() );

  // Now wait one second
  led.cycle( 1000 );
 
  // Blink the same led quickly 
  led.blink( 50, 50 ).trigger( Atm_led::EVT_BLINK );

  // Let it blink for 5 seconds
  led.cycle( 5000 );  

  // And turn it off
  led.trigger( Atm_led::EVT_OFF );
}

void loop() {
}

```

This pattern can be useful when you're starting up your sketch or when - for some reason - you don't need the whole multi-tasking multi-state machine shebang (yet). Look at the sos1 example and compare it to the sos2 and sos3 examples to see why this can sometimes be very efficient.

## Debugging ##


### Machine& Machine::setTrace( Stream* stream, swcb_sym_t callback, const char symbols[] ) ###

Log a machine's state changes and events to the serial Arduino terminal or any other Stream object;

```c++
setTrace( stream, atm_serial_debug::trace, 
          "BIT\0EVT_ON\0EVT_OFF\0EVT_TOGGLE\0EVT_INPUT\0ELSE\0OFF\0ON\0INPUTM" );
```

### Machine variables ###

Variable | Type | Function
-------- | -------- | --------
current | state_t | Holds the numeric value of current state
next | state_t | Holds the numeric value of next state (or -1)
sleep | uint8_t | Value is 1 when the machine is asleep, else 0
cycles | uint32_t | Cycles counted since the last state switch
state_millis | uint32_t | Value of millis() at the last state switch
last_trigger  | uint8_t | The event that triggered the last state switch
next_trigger | uint8_t | Incoming event from the trigger() method

