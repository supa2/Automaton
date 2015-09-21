The idea is to subdivide your programs into subtasks, usually but not exclusively, centered around one or more inputs or outputs and to create a Finite State Machine (or Automaton) for each of them. The machines can interact with each other by setting the other machines' states or sending signals or messages. In my experience this results in modular, easy to understand code that runs extremely stable. In fact most of your application won't even be coded, but described as a table of states that uses pins, timers and counters as input. 

Traditionally, let's start with the simplest possible application: blinking a Led.

* [The skeleton](#the-skeleton)
* [The beginning](#the-beginning)
* [The state table](#the-state-table)
* [The events](#the-events)
* [The actions](#the-actions)
* [Linking it all up](#linking-it-all-up)
* [Admiring the result](#admiring-the-result)
* [Add some debugging](#add-some-debugging)
* [Wrap up](#wrap-up)


### The skeleton ###

To create a new Automaton (state machine) you must first subclass the Automaton base class. You also give it a short class label by which we can recognize it later, when it's up and running.

```c++
#include <Automaton.h>

class Blink : public Machine {

  public:
    Blink( void ) : Machine() { class_label = "BLNK"; };

};

void setup()
{
}


void loop()
{
}
```

### The beginning ###

The Blink subclass needs a new begin() method that sets the class_label and calls the parent class' begin() method, we pass it two arguments, the pin that the led is connected to and the blinkrate in milliseconds. You also need the STATES enum that will contain the Led class's possible states, say LED_ON and LED_OFF.

You need a pin variable to hold the number of the pin the Led object is attached to and when you're at it you can set the pinmode to OUTPUT here. You'll be needing a millisecond timer, so we declare that as well. We'll also need an *EVENTS* enum which for now we fill with just the *ELSE* pseudo-event which must always be the last of the EVENTS.

```c++
class Blink : public Machine {

  public:
    Blink( void ) : Machine() { class_label = "BLNK"; };

    short pin;
    atm_milli_timer timer;

    enum { LED_ON, LED_OFF } STATES;
    enum { ELSE } EVENTS;

    Blink & begin( int attached_pin, int blinkrate )
    {
      pin = attached_pin; 
      set( timer, blinkrate ); 
      pinMode( pin, OUTPUT ); 
      return *this;          
    }
};
```

### The state table ###

The Automaton state machines are table based, so obviously we need a table. This so called *State transition table* has a row for every state the machine can be in. Per state we store the actions that should be taken (the first 3 columns) and the events that must be monitored (the following columns). If a monitored event occurs this will lead to a change in the machine state.

The state table must be defined in our machine's begin() method. The address and the width of the table (that's part of the reason the *ELSE* must be there) are passed to the base class Machine::begin() method.

```c++
    Blink & begin( int attached_pin, int blinkrate )
    {
      const static state_t state_table[] PROGMEM = {
      /*            ON_ENTER    ON_LOOP  ON_EXIT    ELSE */
      /* LED_ON  */       -1,        -1,      -1,     -1,
      /* LED_OFF */       -1,        -1,      -1,     -1,
      };
      Machine::begin( state_table, ELSE );
      pin = attached_pin; 
      set( timer, blinkrate ); 
      pinMode( pin, OUTPUT ); 
      return *this;          
    }
```

Don't forget to declare the state table as static and PROGMEM. The *static* means the table contents will still be there when your machine's begin() method exits, the *PROGMEM* keyword makes sure the data is kept in flash memory which is much more abundant than RAM.

For now the state table consists of two rows (the LED_OFF and LED_ON states we just thought of) and four columns. All fields are filled with the value -1, which means as much as 'do nothing'. And that's exactly what your new machine does - for now.

### The events ###

A state machine changes its state in response to events. The only event we need for our machine is time based, the timer we already declared. Now we expand our EVENTS list with a timer event (let's call it EVT_TIMER) and add an extra column to the state table to accomodate it.


```c++
    enum { LED_ON, LED_OFF } STATES;
    enum { EVT_TIMER, ELSE } EVENTS;

    Blink & begin( int attached_pin, int blinkrate )
    {
      const static state_t state_table[] PROGMEM = {
      /*            ON_ENTER    ON_LOOP  ON_EXIT    EVT_TIMER  ELSE */
      /* LED_ON  */       -1,        -1,      -1,          -1,   -1,
      /* LED_OFF */       -1,        -1,      -1,          -1,   -1,
      };
      Machine::begin( state_table, ELSE );
      pin = attached_pin; 
      set( timer, blinkrate ); 
      pinMode( pin, OUTPUT ); 
      return *this;          
    }

    int event( int id ) 
    {
    }

    void action( int id ) 
    {

    }

```

PLease note that the order of the events in the *EVENTS* enum must be the same as the order of the events in the state table columns (starting with column 4). Similarly, the order of the rows in the state table must be the same as the order of the states in the *STATES* enum. It's very important to update the comments surrounding the state table rows and headers so you stand a chance of understanding what's happening when you look at your code at a later time. 

Each Automaton state machine must define an event() and an action() method so we created two empty ones. The event handler is called by the state machine whenever it needs to know if an event has occured. If it wants to know if the timer we just declared has expired it calls the event() handler with our *EVT_TIMER* value as a parameter. The handler must respond to that request with either a 0 (event did not occur) or 1 (event occurred). We can achieve that with the following code:

```c++
    int event( int id ) 
    {
      switch ( id ) {
        case EVT_TIMER :
          return expired( timer );
      }
      return 0;
    }
```

The expired() method checks the timer against the number of millisecond the state has been active and returns either 0 (still running) or 1 (expired). 

### The actions ###

Events are the inputs of a state machine. Our blink machine only needs one, a timer. It also needs outputs, we call them actions. In the case of our blink machine it needs two actions. One that turns the led on (let's call it *ACT_ON*) and one that turns it off again (*ACT_OFF*).

```c++
    enum { LED_ON, LED_OFF } STATES;
    enum { EVT_TIMER, ELSE } EVENTS;
    enum { ACT_ON, ACT_OFF } ACTIONS;
```
For a change, order isn't important and we also don't need some magic value at the end. To process the actions our machine has an action() method similar to the event() method.

```c++
    void action( int id ) 
    {
      switch ( id ) {
        case ACT_ON :
          digitalWrite( pin, HIGH );
          return;
        case ACT_OFF :
          digitalWrite( pin, LOW );
          return;
       }
    }
```

Whenever the state machine calls action( ACT_ON ) it turns the led on and when it calls action( ACT_OFF) it turns the led off. 

### Linking it all up ###

Writing the begin(), event() and action() method is all the programming we have to do in this case. All we need to get a working state machine is to tell it what we want it to do. We do that by filling in the state transition table. Let's start with the actions. We want the led to switch on when the machine enters the *LED_ON* state (*ON_ENTER*: *ACT_ON*). Find the row that defines the *LED_ON* state and put the value *ACT_ON* in the *ON_ENTER* column. And similarly we want the led to switch off when the machine enters the *LED_OFF* state (*ON_ENTER*: *ACT_OFF*) We fill in that value in the second row. You can also choose to perform an action when the machine exits a certain state or whenever it cycles in that state, but we don't need that for our blink machine.

```c++
      const static state_t state_table[] PROGMEM = {
      /*            ON_ENTER    ON_LOOP  ON_EXIT    EVT_TIMER  ELSE */
      /* LED_ON  */   ACT_ON,        -1,      -1,          -1,   -1,
      /* LED_OFF */  ACT_OFF,        -1,      -1,          -1,   -1,
      };
```

We've now linked our outputs, but we're not done yet. We need to link our inputs as well. Whenever the timer expires we want the machine to switch from *LED_ON* to *LED_OFF* and vice versa. So, when the the machine is in state *LED_ON* we want it to check if the timer has expired and if it has we want the machine to switch to state *LED_OFF*. In an Automaton machine we achieve this by putting *LED_OFF* in the *EVT_TIMER* column in the *LED_ON* row. We do the similar but opposite thing for the *LED_OFF* state and we end up with a finished state table.

```c++
      const static state_t state_table[] PROGMEM = {
      /*            ON_ENTER    ON_LOOP  ON_EXIT    EVT_TIMER  ELSE */
      /* LED_ON  */   ACT_ON,        -1,      -1,     LED_OFF,   -1,
      /* LED_OFF */  ACT_OFF,        -1,      -1,      LED_ON,   -1,
      };
```
### Admiring the result ###

Now all we need to do to admire our gloriously blinking led is to instanciate the class in an object which is just a fancy way of saying...

```c++
Blink led;
```

...initialize our object in the Arduino's setup() function...

```c++
void setup() {
  led.begin( 3, 250 );
}
```

...and call the Blink machine's cycle() method from the Arduino loop:

```c++
void loop() {
  led.cycle();
}
```

And presto, the machine blinks at two blinks per second.

This is the complete code as a single Arduino sketch file

```c++
#include <Automaton.h>

class Blink : public Machine {

  public:
    Blink( void ) : Machine() { class_label = "BLNK"; };

    short pin;
    atm_milli_timer timer;

    enum { LED_ON, LED_OFF } STATES;
    enum { EVT_TIMER, ELSE } EVENTS;
    enum { ACT_ON, ACT_OFF } ACTIONS;

    Blink & begin( int attached_pin, int blinkrate )
    {
      const static state_t state_table[] PROGMEM = {
      /*            ON_ENTER    ON_LOOP  ON_EXIT    EVT_TIMER  ELSE */
      /* LED_ON  */   ACT_ON,        -1,      -1,     LED_OFF,   -1,
      /* LED_OFF */  ACT_OFF,        -1,      -1,      LED_ON,   -1,
      };
      Machine::begin( state_table, ELSE );
      pin = attached_pin; 
      set( timer, blinkrate ); 
      pinMode( pin, OUTPUT ); 
      return *this;          
    }

    int event( int id ) 
    {
      switch ( id ) {
        case EVT_TIMER :
          return expired( timer );
      }
      return 0;
    }
    
    void action( int id ) 
    {
      switch ( id ) {
        case ACT_ON :
          digitalWrite( pin, HIGH );
          return;
        case ACT_OFF :
          digitalWrite( pin, LOW );
          return;
       }
    }
};

Blink led;

void setup() {
  led.begin( 3, 250 );

}

void loop() {
  led.cycle();
}
```

### Add some debugging ###

Blink is a rather trivial machine and it's easy to picture how it works, but sometimes it's nice to be able to look inside a machine object and see it switch states as it happens. There's a hook inside the Machine class that allows just that. Add a callback function that prints information about the current state and modify the setup() function so that a Serial port is opened and the callback function is passed to the onSwitch() method.

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
  Serial.print( " on trigger " );
  Serial.print( trigger );
  Serial.print( " (" );
  Serial.print( cycles );
  Serial.print( " cycles in " );
  Serial.print( runtime );
  Serial.println( " ms)" );
}

void setup() {
  Serial.begin( 9600 );
  led.onSwitch( sw, "LED_ON\0LED_OFF","EVT_TIMER\0ELSE" );
  led.begin( 3, 250 );
}
```

The onSwitch() method takes three arguments. The first is the function to call just before a state switch. The second and third are NULL delimited strings that hold the state and event labels that your machine uses. They must be in the same order as the enums in your class. Now, when you run the tutorial example with the Serial monitor open you'll see something like this:

```
0 Switching BLNK from state *NONE* to LED_ON on trigger *NONE* (1 cycles in 0 ms)
268 Switching BLNK from state LED_ON to LED_OFF on trigger EVT_TIMER (7982 cycles in 250 ms)
547 Switching BLNK from state LED_OFF to LED_ON on trigger EVT_TIMER (7974 cycles in 250 ms)
827 Switching BLNK from state LED_ON to LED_OFF on trigger EVT_TIMER (7974 cycles in 250 ms)
1107 Switching BLNK from state LED_OFF to LED_ON on trigger EVT_TIMER (8007 cycles in 250 ms)
1388 Switching BLNK from state LED_ON to LED_OFF on trigger EVT_TIMER (7972 cycles in 250 ms)
1669 Switching BLNK from state LED_OFF to LED_ON on trigger EVT_TIMER (7972 cycles in 250 ms)
1949 Switching BLNK from state LED_ON to LED_OFF on trigger EVT_TIMER (7973 cycles in 250 ms)
```

You can monitor exactly what machine (class or instance label) switched from one state to another and on which event and at what time (in milliseconds). A very helpful function to help you understand what exactly is going on inside your machines. (it's easy to monitor multiple machines with one callback function)

### Wrap up ###

I can hear some of you saying: "That's a lot of work for just blinking a led! I can do that in two lines!". And of course you can. But the advantage of using state machines is that you can run dozens of state machines at the same time, each performing its own subtask (preferably inside a factory object). The machines can communicate asynchronously via the message queue and you can easily perform complicated tasks while your Arduino stays responsive to user input. Many of today's hottest technologies like Node.JS and Python's twisted are based on event driven frameworks. Now your Arduino can have one too.

For the sake of this tutorial I've kept everything in a single .ino file, but ideally a state machine would be packaged in its own separate .cpp and .h files that can be distributed and shared just like any Arduino library. Look at the source of one of the bundled state machines to see how that's done. The blink example included in the library contains the tutorial source code, the blink_modular example has the same code with the blink machine in separate .cpp/.h files.

