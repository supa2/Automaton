# Automaton Machine class #
----------

*Machine* is an abstract class, which means it cannot be instantiated directly. It must be subclassed first. These subclasses define *Finite State Machines* which inherit all *Machine* functionality and *can* be instantiated (used as an object). Every Machine subclass should define its own version of the *begin*(), *event*() and *action*() methods. 

### begin( tbl, w, [messages], [msg_w] ) ###

Calls the Machine base class initialization code. Links a *state transition table* to the machine and optionally adds an incoming messaging queue if the machine needs to be able to process incoming messages. Each machine subclass should define its own begin() method which, in turn should call Machine::begin() to get the ball rolling.


    Machine::begin( state_table, ELSE );
	Machine::begin( state_table, ELSE, messages, MSG_END );

The *ELSE* and *MSG_END* parameters are identifiers from the *STATES* and *MESSAGES* enum section of the class definition. They are used to determine the dimensions of the data structures.

For this reason each *STATES* enum must end with an *ELSE* event and each *MESSAGES* enum must end with a *MSG_END* entry.

	const static state_t state_table[] PROGMEM = {
	/*                  ON_ENTER    ON_LOOP    ON_EXIT  EVT_INPUT   EVT_EOL   ELSE */
	/* IDLE     */            -1,        -1,        -1,  READCHAR,       -1,    -1,
	/* READCHAR */  ACT_READCHAR,        -1,        -1,  READCHAR,     SEND,    -1,
	/* SEND     */      ACT_SEND,        -1,        -1,        -1,       -1,  IDLE,
	};
	Machine::begin( state_table, ELSE );

The *ELSE* event is automatic (generates no call to the event() method), the *MSG_END* entry is purely used as an end-of-list marker.

### event( id ) ###

Event handler. This is a *pure virtual method* (which means every subclass of Machine must implement it). This handler will be called for every event in the current state where the corresponding table entry is not -1. The id argument contains the event id for which the handler is called.

The Machine class calls the subclass event() method to find out if a certain event (id) has occured. The event() method should return 1 if the requested event has occured and 0 if it hasn't.  

The state machine makes its state change decisions on the basis of what this handler returns combined with the contents of the state table.

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

Failing to implement the event() method in a subclass generates the following compiler error: 

	Cannot declare variable <objectname> to be of abstract type...

### action( id ) ###

Action handler. This is a pure virtual method (which means every subclass of Machine must implement it). This handler will be called for every action in the current state where the corresponding table entry is not -1. The id argument contains the action id for which the handler is called.

	void Atm_led::action( int id )
	{
	  switch ( id ) {
	    case ACT_INIT :
	      set( counter, repeat_count );
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

Failing to implement the action() method in a subclass generates the following compiler error: 

	Cannot declare variable <objectname> to be of abstract type...

*States*
----------

### state( [state] ) ###

Requests the current state of the machine, or if the *state* parameter is set, sets the state the machine will switch to at the start of the next machine cycle.

	if ( led.state() != led.OFF ) {
		led.state( led.OFF );
	}
 

### toggle( state1, state2 ) ###

Toggles the machine's state, equivalent to the following if-else-statement:

	if ( led1.state() == state1 ) {
	     led1.state( state2 );
	} else {
	     led1.state( state1 );
	}
	
Example:

	led1.toggle( LED_IDLE, LED_BLINKON );

*Timers, counters & pins*
----------

### set( timer | counter, value ) ###

Sets a timer or counter struct depending on the argument types passed. Normally used inside the object class.

	atm_milli_timer timer1;
	atm_micro_timer timer2;
	atm_counter counter;
	
	set(  timer1, 1000 ); // Set a timer to expire 1 second from now
	set(  timer2, 100 ); // Set a timer to expire 100 microseconds from now
	set( counter, 10   ); // Set a countdown counter to 10

### decrement( counter ) ###

Decrements the counter unless the counter value is already at 0 or the counter is equal to ATM\_COUNTER\_OFF. Returns the new counter value.

	decrement( counter );

### expired( timer | counter ) ###

Returns true if the timer argument has expired.
Always returns true if the timer was set to 0.
Always returns false if the timer was set to ATM\_TIMER\_OFF.

Returns true if the counter argument has expired (counted down to 0).
Always returns true if the counter was set to 0.
Always returns false if the counter was set to ATM\_COUNTER\_OFF.

	case EVT_TIMER :
	     return expired( _timer );
	case EVT_COUNTER :
	     return expired( _counter );

### pinChange( pin, [hilo] ) ###

Returns true if the pin state has changed from low to high or high to low, optionally, if the hilo argument was specified only returns true if the change was in the specified direction. Always clears any change.

     case EVT_PRESSED :
          return pinChange( pin, LOW );

### milli_runtime() ###

Returns the runtime of the current object state in milliseconds.

	Serial.print( led1.milli_runtime() );

### micro_runtime() ###

Returns the runtime of the current object state in microseconds.

	Serial.print( led1.micro_runtime() );

*Scheduling*
----------

### asleep() ###

Returns true if the object is in sleeping state (which is the case if the current state has the ATM\_SLEEP constant on the ON\_LOOP column). A machine in a sleeping state does not execute its event loop and does not call its action() handler, it does, however, process incoming messages.

	led1.asleep();

### priority( [prio] ) ###

Sets or retrieves the state machine's priority setting. The default priority is 1, which runs the machine at full speed. Priority 2 runs it at half speed. Priority 3 runs at quarter speed. Finally, priority 4 runs at 1/8 speed.

	led1.priority( 2 );

	// Disable button
	button.priority( 0 );

    // Re-enable it
    button.priority( 1 ); 

Set a machine's priority to 0 to disable it altogether, This uses even less resources than sleeping. Incoming messages are not processed in this mode.

### cycle() ###

Executes one cycle of the state machine. Normally only called by the factory class but can also be used directly inside the Arduino loop() function to bypass the factory class altogether. (may be slightly faster if you don't require different machine priorities - see the priority() method)

	void loop()
	{
      led1.cycle();
      led2.cycle();
      led3.cycle();
	}

*Message queue*
----------

The Machine class defines a simulated messaging queue via which messages can be sent from machine to machine or from the main Arduino program to a machine. Multiple messages can be queued.  

### msgWrite( id, [cnt] ) ###

Adds a new message to the machine's message queue. If the *cnt* argument is supplied adds that number of messages to the queue. The available message types (id) are defined in the machine's .h file.


In the .h file:

	enum { MSG_OFF, MSG_ON } MESSAGES;

In the main program:

	obj.msgWrite( obj.MSG_ON );

To allow processing of incoming messages a sleeping machine is woken up by a call to msgWrite(). 
 
### msgRead( id, [cnt] ) ###

Checks the queue for the given message type (id), if one is found removes it from the queue and returns 1. This method is normally used in a machine's event handler.

	case EVT_OFF :
	  return msgRead( MSG_OFF );
	case EVT_ON :
	  return msgRead( MSG_ON );
 
If the *cnt* argument is given removes *cnt* messages from the queue.

### msgPeek( id ) ###

Checks the queue for the given message type (id), if one is found leaves it in the queue and returns 1.

	case EVT_DISABLED :
		return msgPeek( MSG_DISABLED );

This method can be used in conjunction with msgWrite() and msgClear() to simulate setting, checking and clearing a flag.

### msgClear( [id] ) ###

Clears all messages of a certain type (id) from the queue, or, if no id argument, is given flushes the entire queue.

	obj->msgClear( MSG_DISABLED );
	obj->msgClear(); 

### msgMap( bitmap ) ###

Checks the bitmap variable and for each bit set executes a msgWrite() to the corresponding message id.

	obj->msgMap( 4 );

Is equivalent to:

	obj->msgWrite( 0 );
	obj->msgWrite( 2 );

To allow processing of incoming messages a sleeping machine is woken up by a call to msgMap(). 

*Debugging*
----------

### label( inst_label ) ###

Overrides the machine's default (class based) label and sets a new one for the current instance.

	led1->label( "LED_R" );
	led2->label( "LED_G" );
	led3->label( "LED_B" );

This label can be used to access the machine (via the Factory::find() method) or to distinguish the machine from other instances in the same class in log output generated by onSwitch().    

### onSwitch( callback, [sym\_states], [sym\_events] ) ###

Registers a callback which will be called just before a machine status switch. May be used to selectively log machine behavior. 

	void sw( const char label[], int current, int next, 
		int trigger, uint32_t runtime, uint32_t cycles ) {
	  Serial.print( millis() );
	  Serial.print( " Switching " );
	  Serial.print( label );
	  Serial.print( " from state " );
	  Serial.print( current );
	  Serial.print( " to " );
	  Serial.print( next );
	  Serial.print( " on trigger " );
	  Serial.print( trigger );
	  Serial.print( " (" );
	  Serial.print( cycles );
	  Serial.print( " cycles in " );
	  Serial.print( runtime );
	  Serial.println( " ms)" );
	}

	obj->onSwitch( sw ).label( "TST" );


The code above will log numeric values for states and events (triggers) which requires some interpretation. Use the extended version of this method to provide symbol tables for states and events. The symbol tables are strings that contain NULL ('\0') separated lists of identifier names in the same order they occur in the *STATES* & *EVENTS* enums of the machine you want to monitor. 

	void sw( const char label[], const char current[], const char next[], 
		const char trigger[], uint32_t runtime, uint32_t cycles ) {
	  Serial.print( millis() );
	  Serial.print( " Switching " );
	  Serial.print( label );
	  Serial.print( " from state " );
	  Serial.print( current );
	  Serial.print( " to " );
	  Serial.print( next );
	  Serial.print( " on trigger " );
	  Serial.print( trigger );
	  Serial.print( " (" );
	  Serial.print( cycles );
	  Serial.print( " cycles in " );
	  Serial.print( runtime );
	  Serial.println( " ms)" );
	}

	obj->onSwitch( sw, 
		"IDLE\0WAIT\0PULSE", 
		"EVT_TIMER\0EVT_HIGH\0EVT_LOW\0ELSE" )
			.label( "TST" );

This will provide a considerably more understandable log output. Note that the callback functions in the two examples differ in argument types.

