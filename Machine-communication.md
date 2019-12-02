<!-- md-tocify-begin -->
- Triggering machine events
- Calling custom methods
- Inter-machine communication
- Push Connectors
- Pull connectors
- Callbacks
- Lambda Functions
<!-- md-tocify-end -->

Author: Ben Pinhorn (00benallen / 00ben.allen@gmail.com)

### Triggering machine events ###

The preferred way for to communicate with a state machine is the `trigger()` method. With a `trigger()` call you can cause an event to occur inside a state machine. For this to work the machine must actually be listening to the event (the event's column in the current state must be greater than -1). 

The trigger call cycles a machine at least once before triggering the event and cycles it twice after triggering the event to allow the event to be picked up and processed by the machine. If after the initial cycle the machine is not receptive (not listening to the triggered event or still waiting to process a previous trigger) the trigger method will cycle the machine up to 8 times waiting for it to become responsive. If it still isn't ready by the 8th cycle the event is discarded.

Events can be triggered by a direct call to the state machine's `trigger()` method:

```c++

// Triggering an event
led.begin( 4 ).trigger( led.EVT_ON );

```

### Calling custom methods ###

```c++

// Call the led machine's custom on() method
led.on(); // Note: the Atm_led class doesn't really have an on method

```
The custom method has all freedom in processing the call and can switch state, `trigger()` an event or change a machine variable.

### Inter-machine communication

The above method is designed for the owning code of a `Machine` (like the sketch file of an Arduino project) to send events down into the `Machine`. This is useful for timing machines in sequence, or starting/stopping machines based on non-automaton code. However this method should not be used to send events from one Machine to another. Doing so may or may not work, and could actually crash your project (will likely show up a silent reboot of your Arduino). If you ever find yourself trying to store an instance of a `Machine` INSIDE of another `Machine`, you're generally making a mistake and losing the value of decoupling your `Machine` that this framework offers.

There are two common and powerful ways to communicate between machines, Push Connectors and Pull Connectors.

### Push Connectors

Push connectors follow a classic reactive pattern for `Machines` to send events/information to each other. It works by setting up one `Machine` as a "listener" on a connection, and then the other `Machine` can push events/information through the connection.

Here's a simple example:

```c++

Atm_led led;
Atm_button button;

void setup() {

    button.begin( 2 )
    .onPress( led, led.EVT_ON );

}

// rest of code
```

What's happening here is the `led` Machine is being set a listener for the `ON_PRESS` connection of the `button` Machine.

Here's how the button would push data through this `ON_PRESS` connection:

```c++

void Atm_button::action( int id ) {
  switch ( id ) {
    case ENT_PRESS:
      push( connectors, ON_PRESS, 0, 0, 0);
      return;
    // rest of action function
  }
}

```

`Machine::push` is a function that all `Machines` inherit from the Machine base class. To explain better how it works, here's its definition (simplified)

```c++

/*
 * Machine::push( connectors, id, sub, v, up ) - Pushes an action through the specified connector
 *
 * connectors Connector table
 * id         Connector id
 * sub        Connector sub id (for multi-slot connectors)
 * v          Value to pass to a callback as 'v'
 * up         Value to pass to a callback as 'up'
 *
 */

void Machine::push( atm_connector connectors[], int id, int sub, int v, int up )

```

As you can see, you must pass `push` 5 parameters:
1. An array of connectors, these connectors are the types of connection your Machine supports (more on this later)
2. id of specific connector you want to push to (like `ON_PRESS`)
3. sub id of connector for multi-slot connectors
4. 2 values, labelled `v` and `up`, these can be anything you want. These values are a way to send integer data through connections

The final steps to setting up a connection is to the define the connectors you want to support, and setting up functions to register connections.

To add a new connector to your `Machine`, you just need to add an enum of ids to your class definition, and an array of `atm_connector`s:
```c++
class Atm_button: public Machine {

 public:
  // public stuff for button

 private:
  enum { ON_PRESS, CONN_MAX }; // CONNECTOR IDs
  atm_connector connectors[CONN_MAX]; // array of connectors
  // private stuff for button

};
```
Note: `CONN_MAX` being at the end of the `CONNECTORS` enum is important to keep the connectors array always the right size

Then you can write functions for connection registration
```c++
/*
   onPress() push connector variants ( slots 1, autostore 0, broadcast 0 )
*/

Atm_button& Atm_button::onPress( Machine& machine, int event ) {
  onPush( connectors, ON_PRESS, 0, 1, 1, machine, event );
  return *this;
}

Atm_button& Atm_button::onPress( atm_cb_push_t callback, int idx ) {
  onPush( connectors, ON_PRESS, 0, 1, 1, callback, idx );
  return *this;
}
```
The first function is the variant you saw used above, it allows you to register a Machine to connect, and the event you want to trigger when the machine hosting the connector pushes.

The second function is to allow you to register callbacks to the connection, which you'll see later.

### Pull connectors

Pull connectors are another way of communicating between two machines. The way pull connectors work is backwards to push connectors, the host of the pull connection will query the .state() value of other machines when it calls pull(). The Atm_controller machine is a good example of a machine which makes heavy use of pull connections. 

This type of connection is not commonly used, and breaks the "reactive" part of the Automaton style. If you'd like to learn more about it, look at how the Atm_controller class works, but as of now the Machine Editor doesn't even allow you to create these kinds of connections. 

Consider pull connectors to be an advanced technique, and generally you should be able to connect your machines with Push Connectors only.

### Callbacks

Another way to respond to the data a Machine pushes through its connectors is to assign a callback to that connection. 

```c++
Atm_button button; 

void button_change( int idx, int v, int up ) {
  if ( v == CONSTANT) {
    // Do something when the button is pressed
  }
}

void setup() {
  button.begin( 2 )
    .onPress( button_change );
}

// rest of project
```

This variant of onPress allows the owner of the Machine to do whatever they want with the `v` and `up` values.

### Lambda Functions
C++14 added lambda functions to the language, see [Microsoft's Lamdba Functions Guide](https://docs.microsoft.com/en-us/cpp/cpp/lambda-expressions-in-cpp?view=vs-2019) for details on how these work.

Lambda functions are basically unnamed/anonymous functions you can define on the fly. Here's a basic example of using these for connections:
```c++
Atm_custom_machine machine

void setup() {
    machine.onChange(
    [] (int idx, int v, int up) { // lambda function
    switch (v) {
      case 1:
        if (up) {
            // do something
        }
        return;
      case 2:
        // do something else
        return;
    });
}
```

This syntax allows easy definitions of complex data handling when Machines push data through their connections. 

A common use case for callbacks/lambda functions is to trigger multiple different events on another Machine when `push` is called. Here's an example with two hypothetical Machines. One handles four DC motors, and the other a joystick.

```c++
Atm_motors motors(3, 4, 2, 1);
Atm_joystick joystick(1);

void setup() {

    joystick.onChange([] (int idx, int v, int up) {
        switch (v) {
            case Atm_joystick::MOTOR_LEFT:
                motors.left();
                return;
            case Atm_joystick::MOTOR_RIGHT:
                motors.right();
                return;
            case Atm_joystick::MOTOR_FORWARD:
                motors.forward();
                return;
            case Atm_joystick::MOTOR_STOP:
                motors.stop();
                return;
        }
    });

}
```
Reminder: the `motors.left()`, `motors.right()` functions are custom functions for calling `trigger` on the motors.

Using this lamdba function, we've been able to use v to trigger one of four different events on the motors Machine using a single connection.

Here's what the internals of Atm_joystick would look like to enable this kind of behaviour
```c++
// Header file

class Atm_joystick: public Machine {
    // implementation of joystick
    private: 
        enum { MOTOR_LEFT, MOTOR_RIGHT, MOTOR_STOP, MOTOR_FORWARD }
        enum { ON_CHANGE, CONN_MAX }; // connection ids
        atm_connector connectors[CONN_MAX]; // connection array
};

// cpp file

// rest of code

/* Add C++ code for each action
   This generates the 'output' for the state machine

   Available connectors:
     push( connectors, ON_CHANGE, 0, <v>, <up> );
*/

void Atm_joystick::action( int id ) {
  switch ( id ) {
    case ENT_LEFT:
      push ( connectors, ON_CHANGE, 0, MOTOR_LEFT, 0);
      return;
    case ENT_RIGHT:
      push ( connectors, ON_CHANGE, 0, MOTOR_RIGHT, 0);
      return;
    case ENT_UP:
      push ( connectors, ON_CHANGE, 0, MOTOR_FORWARD, 0);
      return;
    case ENT_DOWN:
      push ( connectors, ON_CHANGE, 0, MOTOR_STOP, 0);
      return;
  }
}
```
And that's it! Now the `Atm_joystick` `Machine` can control the `Atm_motors` machine's internal state in a decoupled way.

### Conclusions

I hope this part of the guide is a complete enough tutorial on how to connect your Automatons together and get the fully utility out of this framework. 

If there's anything you think is missing or would like to add, feel free to open an issue!

