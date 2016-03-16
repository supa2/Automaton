- Using the message queue
- Triggering state events
- Calling custom methods
- Calling state() method

### Using the message queue ###

```c++

// Sending messages:
led.msgWrite( led.MSG_ON );
factory.msgSend( 'LED', led.MSG_ON );

// Receiving messages
msgRead( MSG_ON );

```

This is the highest level of machine-to-machine communication. Messages can be sent directly to a machine object or via the factory message queue. When using the factory msgSend() method messages can be broadcast to multiple machines with the same class or instance label.

Messages are read from the machine's event() method and handled according to the state transition table.

### Triggering state events ###

```c++

// Triggering an event
led.trigger( led.EVT_ON );

```
Directly changes the target machine to the new state matching the event and current state indicated in the state transition table. The state transition table determines how (and if) an event is processed.

### Calling custom methods ###

```c++

// Call the led machine's custom on() method
led.on();

```
The custom method has all freedom in processing the call and can switch state, trigger() an event or change a machine variable.

### Calling state() method ###

```c++

// Directly change the led machine's state
led.state( led.ON );

```

This bypasses the machine's internal state logic and changes the state in a manner not necessarily foreseen by the machine's creator. Only use from outside the machine if you know what you're doing.