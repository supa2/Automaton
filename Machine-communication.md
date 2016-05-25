<!-- md-tocify-begin -->
- Push Connectors
- Pull connectors
- Callbacks
+ Lambda functions (closures)  
- trigger()
- state()
- Calling custom methods

<!-- md-tocify-end -->

### Triggering machine events ###

The preferred way for state machines to communicate with each other is the trigger() method. With a trigger() call you can cause an event to occur inside a state machine. For this to work the machine must actually be listening to the event (the event's column in the current state must be greater than -1). 

The trigger call cycles a machine at least once before triggering the event and cycles it twice after triggering the event to allow the event to be picked up and processed by the machine. If after the initial cycle the machine is not receptive (not listening to the triggered event or still waiting to process a previous trigger) the trigger method will cycle the machine up to 8 times waiting for it to become responsive. If it still isn't ready by the 8th cycle the event is discarded.

Events can be triggered by a direct call to the state machine's trigger() method:

```c++

// Triggering an event
led.begin( 4 ).trigger( led.EVT_ON );

```

### Calling custom methods ###

```c++

// Call the led machine's custom on() method
led.on(); // Note: the Atm_led class doesn't really have an on method

```
The custom method has all freedom in processing the call and can switch state, trigger() an event or change a machine variable.

