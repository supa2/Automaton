- Using the message queue
- Triggering state events
- Calling state() method
- Calling custom methods

### Using the message queue ###

```c++

led.msgWrite( led.MSG_ON );

```

### Triggering state events ###

```c++

led.trigger( led.EVT_ON );

```

### Calling state() method ###

```c++

led.state( led.ON );

```

### Calling custom methods ###

```c++

led.on();

```
