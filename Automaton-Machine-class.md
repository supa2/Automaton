*Machine* is an abstract class, which means it cannot be instantiated directly. It must be subclassed first. These subclasses define *Finite State Machines* which inherit all *Machine* functionality and *can* be instantiated (used as an object). Every Machine subclass should define its own version of the *begin*(), *event*() and *action*() methods. 

* [Initialization](#initialization)
 * begin()
 * event()
 * action()
* [States](#states)
 * state()
 * trigger()
* [Timers & pins](#user-content-timers--pins)
 * pinChange() *
 * runtime_millis()
 * runtime_micros()
* [Scheduling](#scheduling)
 * asleep()
 * priority() *
 * cycle()
* [Message queue](#message-queue)
 * msgQueue() *
 * msgWrite() *
 * msgRead() *
 * msgPeek() *
 * msgClear() *
* [Debugging](#debugging)
 * label() *
 * onSwitch() *

(methods marked with an asterisk * are not available in a TinyMachine subclass)

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
/* READCHAR */  ACT_READCHAR,        -1,        -1,  READCHAR,     SEND,    -1,
/* SEND     */      ACT_SEND,        -1,        -1,        -1,       -1,  IDLE,
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

Action handler. This is a pure virtual method (which means every subclass of Machine must implement it). This handler will be called for every action in the current state where the corresponding table entry is not -1. The id argument contains the action id for which the handler is called.

```c++
void Atm_led::action( int id )
{
  switch ( id ) {
    case ACT_INIT :
      counter.set( repeat_count );
      digitalWrite( pin, LOW );
      return;
    case ACT_ON :
      decrement( counter );
      digitalWrite( pin, HIGH );
      return;
    case ACT_OFF :
      digitalWrite( pin, LOW );
      return;
  }
}
```

Just before every state change (and just before the onSwitch() callback, see below) the state machine calls the action handler with the *ATM_ON_SWITCH* id. In that case the 'next' variable contains the new state and the 'current' variable contains the old (present) state.

Failing to implement the action() method in a subclass generates the following compiler error: 

```
Cannot declare variable <objectname> to be of abstract type...
```

## States ##

### Machine & state( state_t state) ###

Requests the current state of the machine, or if the *state* parameter is set, sets the state the machine will switch to at the start of the next machine cycle.

```c++
if ( led.state() != led.OFF ) {
	led.state( led.OFF );
}
```

It's considered somewhat bad practice to set a machine's state directly from the outside because you're in fact bypassing the state transition table. Use of the trigger() or messaging methods is preferred.

### int trigger( int event ) ###

Triggers an event for the current state. If there's a positive number in the event column for the current state the machine will switch to that state on the next cycle. The method will return 1 if the trigger has resulted in a state change or 0 if it hasn't.


```c++
void setup() {
  led1.begin( 4 );
  factory.add( led1 );
  led1.trigger( led1.EVT_BLINK );
}
```

Note that the machine being triggered must have been initialized which means it must have been cycled at least once since the call to begin(). The factory.add() methods will automatically cycle each added machine once so that it will have been initialized. This can also be done explicitely like this in case you don't use Factory:

```c++
  led1.begin( 4 );
  led1.cycle().trigger( led1.EVT_BLINK );
```

The trigger() method is a lightweight alternative to the message queue (which uses SRAM). Triggered events can not be stored but are processed immediately or discarded.

## Timers & pins ##

### uint8_t pinChange( uint8_t pin ) ###

Returns true if the pin state has changed from low to high or high to low. Always clears any change.

```c++
case EVT_CHANGED :
     return pinChange( pin );
```

### uint32_t runtime_millis( void ) ###

Returns the runtime of the current object state in milliseconds.

```c++
Serial.print( led1.runtime_millis() );
```

### uint32_t runtime_micros( void ) ###

Returns the runtime of the current object state in microseconds.

```c++
Serial.print( led1.runtime_micros() );
```

More about timers and counters: [here](https://github.com/tinkerspy/Automaton/wiki/Timers-&-Counters)

## Scheduling ##
----------

### uint8_t asleep( void ) ###

Returns true if the object is in sleeping state (which is the case if the current state has the ATM\_SLEEP constant on the ON\_LOOP column). A machine in a sleeping state does not execute its event loop and does not call its action() handler, it does, however, process incoming messages.

```c++
led1.asleep();
```

### Machine & priority( int8_t priority ) ###

Sets or retrieves the state machine's priority setting. The default priority is 1, which runs the machine at full speed. Priority 2 runs it at half speed. Priority 3 runs at quarter speed. Finally, priority 4 runs at 1/8 speed.

Priority | Speed | Machine cycles per factory cycle
------------ | ------------- | -------------:
0 | 0% | 0
1 | 100% | 8
2 | 50% | 4
3 | 25% | 2
4 | 12.5% | 1

```c++
led1.priority( 2 );

// Disable button
button.priority( 0 );

// Re-enable it
button.priority( 1 ); 
```

The use of the term speed may be confusing. If a led blinking machine runs at priority 2, that doesn't mean the led will blink twice as slow. It means it won't be updated as often. Which in the case of a blinking led probably won't be noticable.

Set a machine's priority to 0 to disable it altogether, This uses even less resources than sleeping. Incoming messages are not processed in this mode.

### Machine & cycle( void ) ###

Executes one cycle of the state machine. Normally only called by the factory class but can also be used directly inside the Arduino loop() function to bypass the factory class altogether. (may be slightly faster if you don't require different machine priorities - see the priority() method)

```c++
void loop()
{
    led1.cycle();
    led2.cycle();
    led3.cycle();
}
```

## Message queue ##

The Machine class defines a simulated messaging queue via which messages can be sent from machine to machine or from the main Arduino program to a machine. Multiple messages can be queued.  

### Machine & msgQueue( atm_msg_t msg[], int width [, int auto_clear] )  ###

The msgQueue() methods adds an incoming messaging queue if the machine needs to be able to process incoming messages.

In the Atm_xxx.h file:

```c++
enum { MSG_OFF, MSG_ON, MSG_END } MESSAGES;

atm_msg_t messages[MSG_END];
```

In the Atm_xxx.cpp file:

```c++
Machine::begin( state_table, ELSE );
Machine::msgQueue( messages, MSG_END );
```

You may now send messages to the machine object like this:

```c++
obj.msgWrite( obj.MSG_OFF );
obj.MsgWrite( obj.MSG_ON );
```

And process them in the machine object's event() handler like this:

```c++
switch ( id ) {
  case EVT_OFF :
		  return msgRead( MSG_OFF );
	case EVT_ON :
		  return msgRead( MSG_ON );
}
```

The *MSG_END* identifier must always be last in the list because it is used to determine the size of the msg queue.
If the *autoclear* parameter is set the state machine will automatically clear the message queue on every state switch. It's often a good idea to set this to 1 to avoid common pitfalls in message handling. If you want to keep messages between state switches (in some cases that's useful) set it to 0 or leave it out altogether. Default value is 0 for backwards compatibility.



### Machine & msgWrite( uint8_t id_msg, [int cnt] ) ###

Adds a new message to the machine's message queue. If the *cnt* argument is supplied adds that number of messages to the queue. The available message types (id) are defined in the machine's .h file.


In the .h file:

```c++
enum { MSG_OFF, MSG_ON } MESSAGES;
```

In the main program:

```c++
obj.msgWrite( obj.MSG_ON );
```

To allow processing of incoming messages a call to msgWrite() wakes up a sleeping machine for the duration of one cycle. 
 
### int msgRead( uint8_t id_msg, [int cnt], [int clear] ) ###

Checks the queue for the given message type (id), if one is found removes it from the queue and returns 1. This method is normally used in a machine's event handler.

```c++
case EVT_OFF :
  return msgRead( MSG_OFF );
case EVT_ON :
  return msgRead( MSG_ON );
```
 
If the *cnt* argument is given removes *cnt* messages from the queue. (default: 1)

If the *clear* argument is given clears the entire message queue of all messages.

Note that the Automaton's messaging queue buffers messages. Two consecutive calls to msgWrite( MSG_ON ) will lead two calls to msgRead( MSG_ON ) to return true. If that's not what you want you may use msgClear( MSG_ON ) as a replacement, this will return true just a single time.

### int msgPeek( uint8_t id_msg ) ###

Checks the queue for the given message type (id), if one is found the number of messages is returned. The queue is left unchanged.

```c++
case EVT_DISABLED :
	return msgPeek( MSG_DISABLED );
```
This method can be used in conjunction with msgWrite() and msgClear() to simulate setting, checking and clearing a flag.

### int msgClear( uint8_t id_msg ) ###
Alternative: Machine & msgClear()  

Clears all messages of a certain type (id) from the queue, or, if no id argument, is given flushes the entire queue.

```c++
obj.msgClear( MSG_DISABLED );
obj.msgClear(); 
```

When the id argument is given returns 1 when one or more messages were present or 0 if there was none. This allows msgClear() to be used instead of msgRead() if the latter's message buffering is not wanted.

## Debugging ##

### Machine & label( const char label[] ) ###

Overrides the machine's default (class based) label and sets a new one for the current instance.

```c++
led1.label( "LED_R" );
led2.label( "LED_G" );
led3.label( "LED_B" );
```

This label can be used to access the machine (via the Factory::find() method) or to distinguish the machine from other instances in the same class in log output generated by onSwitch().    

### Machine & onSwitch( swcb_num_t callback ) ###
### Machine & onSwitch( swcb_sym_t callback, const char sym_s[], const char sym_e[] ) ###

Registers a callback which will be called just before a machine status switch. May be used to selectively log machine behavior. 

```c++
void sw( const char label[], int current, int next, 
	int trigger, uint32_t runtime, uint32_t cycles ) {
  Serial.print( millis() );
  Serial.print( " Switching " );
  Serial.print( label );
  Serial.print( " from state " );
  Serial.print( current );
  Serial.print( " to " );
  Serial.print( next );
  Serial.print( " on event " );
  Serial.print( trigger );
  Serial.print( " (" );
  Serial.print( cycles );
  Serial.print( " cycles in " );
  Serial.print( runtime );
  Serial.println( " ms)" );
}
```

And in the setup() function:

```c++
obj.onSwitch( sw ).label( "TST" );
```

The code above will log numeric values for states and events (triggers) which requires some interpretation. Use the extended version of this method to provide symbol tables for states and events. The symbol tables are strings that contain NULL ('\0') separated lists of identifier names in the same order they occur in the *STATES* & *EVENTS* enums of the machine you want to monitor. 

```c++
void sw( const char label[], const char current[], const char next[], 
	const char trigger[], uint32_t runtime, uint32_t cycles ) {
  Serial.print( millis() );
  Serial.print( " Switching " );
  Serial.print( label );
  Serial.print( " from state " );
  Serial.print( current );
  Serial.print( " to " );
  Serial.print( next );
  Serial.print( " on event " );
  Serial.print( trigger );
  Serial.print( " (" );
  Serial.print( cycles );
  Serial.print( " cycles in " );
  Serial.print( runtime );
  Serial.println( " ms)" );
}
```

And in the setup() function:

```c++
obj.onSwitch( sw, 
	"IDLE\0WAIT\0PULSE", 
	"EVT_TIMER\0EVT_HIGH\0EVT_LOW\0ELSE" )
		.label( "TST" );
```

This will provide a considerably more understandable log output. Note that the callback functions in the two examples differ in argument types.

