- Using the message queue
- Triggering state events
- Calling state() method
- Calling custom methods

### Using the message queue ###

```c++

// Sending messages:
led.msgWrite( led.MSG_ON );
factory.msgSend( 'LED', led.MSG_ON );

// Receiving messages
msgRead( MSG_ON );

```

### Triggering state events ###

```c++

// Triggering an event
led.trigger( led.EVT_ON );

```

### Calling state() method ###

```c++

// Directly change the led machine's state
led.state( led.ON );

```

### Calling custom methods ###

```c++

// Call the led machine's on() method
led.on();

```
