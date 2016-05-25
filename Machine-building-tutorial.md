This tutorial shows you how to build your own state machine within the Automaton framework.

The idea is to subdivide your programs into subtasks, usually but not exclusively, centered around one or more inputs or outputs and to create a Finite State Machine (or Automaton) for each of them. The machines can interact with each other by triggering each others events. In my experience this results in modular, easy to understand code that runs extremely stable. In fact most of your application won't even be coded, but described as a table of states that uses pins, timers and counters as input. 

Traditionally, let's start with the simplest possible application: blinking a Led.

<!-- md-tocify-begin -->
* [The skeleton](#the-skeleton)  
* [The beginning](#the-beginning)  
* [The state table](#the-state-table)  
* [The events](#the-events)  
* [The actions](#the-actions)  
* [Linking it all up](#linking-it-all-up)  
* [Admiring the result](#admiring-the-result)  
* [Adding external events](#adding-external-events)  
* [Full source code](#full-source-code)  
* [Add some debugging](#add-some-debugging)  
* [Wrap up](#wrap-up)  

<!-- md-tocify-end -->


### The skeleton ###

To create a new Automaton (state machine) you must first subclass the Automaton base class. You also give it a name (convention is that Automaton machine names start with Atm_).

```c++
#include <Automaton.h>

class Atm_blink : public Machine {

  public:
    Atm_blink( void ) : Machine(){};

};

void setup() {
}


void loop() {
}
```

### The beginning ###

The Blink subclass needs a new begin() method that calls the parent class' begin() method, we pass it two arguments, the pin that the led is connected to and the blinkrate in milliseconds. You also need the STATES enum that will contain the Led class's possible states, say LED_ON and LED_OFF.

You need a pin variable to hold the number of the pin the Led object is attached to and when you're at it you can set the pinmode to OUTPUT here. You'll be needing a millisecond timer, so we declare that as well. We'll also need an *EVENTS* enum which for now we fill with just the *ELSE* pseudo-event which must always be the last of the EVENTS.

```c++
class Atm_blink : public Machine {

  public:
    Atm_blink( void ) : Machine() {};

    short pin;
    atm_timer_millis timer;

    enum { LED_ON, LED_OFF }; // STATES
    enum { ELSE }; // EVENTS

    Atm_blink & begin( int attached_pin, int blinkrate ) {
      pin = attached_pin; 
      timer.set( blinkrate ); 
      pinMode( pin, OUTPUT ); 
      return *this;          
    }
};
```

### The state table ###

The Automaton state machines are table based, so obviously we need a table. This so called *State transition table* has a row for every state the machine can be in. Per state we store the actions that should be taken (the first 3 columns) and the events that must be monitored (the following columns). If a monitored event occurs this will lead to a change in the machine state.

The state table must be defined in our machine's begin() method. The address and the width of the table (that's part of the reason the *ELSE* must be there) are passed to the base class Machine::begin() method.

```c++
    Atm_blink & begin( int attached_pin, int blinkrate ) {
      const static state_t state_table[] PROGMEM = {
      /*            ON_ENTER    ON_LOOP  ON_EXIT    ELSE */
      /* LED_ON  */       -1,        -1,      -1,     -1,
      /* LED_OFF */       -1,        -1,      -1,     -1,
      };
      Machine::begin( state_table, ELSE );
      pin = attached_pin; 
      timer.set( blinkrate ); 
      pinMode( pin, OUTPUT ); 
      return *this;          
    }
```

Don't forget to declare the state table as static and PROGMEM. The *static* means the table contents will still be there when your machine's begin() method exits, the *PROGMEM* keyword makes sure the data is kept in flash memory which is much more abundant than RAM.

For now the state table consists of two rows (the LED_OFF and LED_ON states we just thought of) and four columns. All fields are filled with the value -1, which means as much as 'do nothing'. And that's exactly what your new machine does - for now.

### The events ###

A state machine changes its state in response to events. The only event we need for our machine is time based, the timer we already declared. Now we expand our EVENTS list with a timer event (let's call it EVT_TIMER) and add an extra column to the state table to accommodate it.


```c++
    enum { LED_ON, LED_OFF }; // STATES
    enum { EVT_TIMER, ELSE }; // EVENTS

    Atm_blink & begin( int attached_pin, int blinkrate ) {
      const static state_t state_table[] PROGMEM = {
      /*            ON_ENTER    ON_LOOP  ON_EXIT    EVT_TIMER  ELSE */
      /* LED_ON  */       -1,        -1,      -1,          -1,   -1,
      /* LED_OFF */       -1,        -1,      -1,          -1,   -1,
      };
      Machine::begin( state_table, ELSE );
      pin = attached_pin; 
      timer.set( blinkrate ); 
      pinMode( pin, OUTPUT ); 
      return *this;          
    }

    int event( int id )  {
    }

    void action( int id ) {

    }

```

PLease note that the order of the events in the *EVENTS* enum must be the same as the order of the events in the state table columns (starting with column 4). Similarly, the order of the rows in the state table must be the same as the order of the states in the *STATES* enum. It's very important to update the comments surrounding the state table rows and headers so you stand a chance of understanding what's happening when you look at your code at a later time. 

Each Automaton state machine must define an event() and an action() method so we created two empty ones. The event handler is called by the state machine whenever it needs to know if an event has occured. If it wants to know if the timer we just declared has expired it calls the event() handler with our *EVT_TIMER* value as a parameter. The handler must respond to that request with either a 0 (event did not occur) or 1 (event occurred). We can achieve that with the following code:

```c++
    int event( int id ) {
      switch ( id ) {
        case EVT_TIMER :
          return timer.expired( this );
      }
      return 0;
    }
```

The expired() method checks the timer against the number of millisecond the state has been active and returns either 0 (still running) or 1 (expired). 

### The actions ###

Events are the inputs of a state machine. Our blink machine only needs one, a timer. It also needs outputs, we call them actions. In the case of our blink machine it needs two actions. One that turns the led on (let's call it *ACT_ON*) and one that turns it off again (*ACT_OFF*).

```c++
    enum { LED_ON, LED_OFF }; // STATES
    enum { EVT_TIMER, ELSE }; // EVENTS
    enum { ACT_ON, ACT_OFF }; // ACTIONS
```
For a change, order isn't important and we also don't need some magic value at the end. To process the actions our machine has an action() method similar to the event() method.

```c++
    void action( int id ) {
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

Writing the begin(), event() and action() method is all the programming we have to do in this case. All we need to add to get a working state machine is to tell it what we want it to do. We do that by filling in the state transition table. Let's start with the actions. We want the led to switch on when the machine enters the *LED_ON* state (*ON_ENTER*: *ACT_ON*). Find the row that defines the *LED_ON* state and put the value *ACT_ON* in the *ON_ENTER* column. And similarly we want the led to switch off when the machine enters the *LED_OFF* state (*ON_ENTER*: *ACT_OFF*) We fill in that value in the second row. You can also choose to perform an action when the machine exits a certain state (*ON_EXIT* column) or whenever it cycles in that state (*ON_LOOP* column), but we don't need that for our blink machine.

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

Now all we need to do to admire our gloriously blinking led is to instantiate the class in an object which is just a fancy way of saying...

```c++
Atm_blink led;
```

...initialize our object in the Arduino's setup() function...

```c++
void setup() {
  led.begin( 4, 250 ); // Pin 4, blinkrate 250 ms
}
```

...and call the Atm_blink machine's cycle() method from the Arduino loop:

```c++
void loop() {
  led.cycle();
}
```

And presto, the machine blinks at two blinks per second.

### Adding external events ###

So our blinking machine works but perhaps it's just a bit too simple. We can't even turn it on or off with a button. To allow the machine to respond to external events we must extend it. First off all we add a state that allows the machine to be idle, to do nothing. By convention this is usually the first state, state 0. We add a new state named *IDLE* to the states enum and add a new row to the state transition table. The ON_ENTER column of the new state turns the led off by calling the ACT_OFF action. Make sure that the order of the states in the state table is the same as the order of the STATES enum. IDLE comes first in both cases.

```c++
      enum { IDLE, LED_ON, LED_OFF }; // STATES

      const static state_t state_table[] PROGMEM = {
      /*            ON_ENTER    ON_LOOP  ON_EXIT    EVT_TIMER  ELSE */
      /* IDLE    */  ACT_OFF,        -1,      -1,          -1,   -1,
      /* LED_ON  */   ACT_ON,        -1,      -1,     LED_OFF,   -1,
      /* LED_OFF */  ACT_OFF,        -1,      -1,      LED_ON,   -1,
      };
```

We need two new events, EVT_ON for turning the machine on and another named EVT_OFF. Each of these events need a new column in the state transition table. Again make sure you maintain the same order in the EVENTS enum and the state table. Link the EVT_ON event in the IDLE state to the LED_ON state so that an incoming EVT_ON trigger will turn the blinker on. Similarly link the EVT_OFF columns in the other states to the IDLE state.

```c++
      enum { IDLE, LED_ON, LED_OFF }; // STATES
      enum { EVT_TIMER, EVT_ON, EVT_OFF, ELSE }; // EVENTS

      const static tiny_state_t state_table[] PROGMEM = {
        /*            ON_ENTER    ON_LOOP  ON_EXIT  EVT_TIMER  EVT_ON  EVT_OFF  ELSE */
        /* IDLE    */  ACT_OFF,        -1,      -1,        -1, LED_ON,      -1,   -1,
        /* LED_ON  */   ACT_ON,        -1,      -1,   LED_OFF,     -1,    IDLE,   -1,
        /* LED_OFF */  ACT_OFF,        -1,      -1,    LED_ON,     -1,    IDLE,   -1,
      };
```

That's all. Now when you fire up the machine you'll notice the led won't blink anymore. It needs to be triggered before it does anything. Add a trigger command to the setup method and you're done.

```c++
void setup() {
  led.begin( 4, 250 ); // Pin 4, blinkrate 250 ms
  led.trigger( led.EVT_ON );
}
```

Now the blinking led can be turned off by an EVT_OFF trigger and turned on again by an EVT_ON trigger. You might have these events triggered by two buttons like this.

```c++
  on_button.begin( 2 ).onPress( led, led.EVT_ON ); // ON button on pin 2
  off_button.begin( 3 ).onPress( led, led.EVT_OFF ); // OFF button on pin 3
```

### Full source code ###

This is the complete Atm_blink code as a single Arduino sketch file

```c++
#include <Automaton.h>

class Atm_blink : public Machine {

  public:
    Atm_blink( void ) : Machine() {};

    short pin;     
    atm_timer_millis timer;

    enum { IDLE, LED_ON, LED_OFF }; // STATES
    enum { EVT_TIMER, EVT_ON, EVT_OFF, ELSE }; // EVENTS
    enum { ACT_ON, ACT_OFF }; //ACTIONS
		
    Atm_blink & begin( int attached_pin, uint32_t blinkrate )
    {
      const static state_t state_table[] PROGMEM = {
      /*            ON_ENTER    ON_LOOP  ON_EXIT  EVT_TIMER  EVT_ON  EVT_OFF  ELSE */
      /* IDLE    */  ACT_OFF,        -1,      -1,        -1, LED_ON,      -1,   -1,
      /* LED_ON  */   ACT_ON,        -1,      -1,   LED_OFF,     -1,    IDLE,   -1,
      /* LED_OFF */  ACT_OFF,        -1,      -1,    LED_ON,     -1,    IDLE,   -1, 
      }; 
      Machine::begin( state_table, ELSE );
      pin = attached_pin; 
      timer.set( blinkrate ); 
      pinMode( pin, OUTPUT ); 
      return *this;          
    }

    int event( int id ) {
      switch ( id ) {
        case EVT_TIMER :
          return timer.expired( this );        
       }
       return 0;
    }
	
    void action( int id ) {
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

Atm_blink led;

void setup() {
  led.begin( 4, 200 );        // Setup a blink machine
  led.trigger( led.EVT_ON );  // Turn it on
}

void loop() {
  led.cycle();
}
```

The blink_modular example, which is based on this tutorial is an excellent staring point to use as a template for building your own machine. Just save a copy of the example under a new name. Rename the .cpp and .h files to something like Atm_mymachine, rename the class and you're well on your way.

### Add some debugging ###

Atm_blink is a rather trivial machine and it's easy to picture how it works, but sometimes it's nice to be able to look inside a machine object and see it switch states as it happens. Enable your new machine to send state change messages by adding the following method to the class:

```c++
Atm_blink & Atm_blink::trace( Stream & stream ) {
  Machine::setTrace( &stream, atm_serial_debug::trace,
    "BLINK\0EVT_TIMER\0EVT_ON\0EVT_OFF\0ELSE\0IDLE\0LED_ON\0LED_OFF" );
  return *this;
}
```
The Machine::setTrace() method takes three arguments, the third is a string starting with the class name and then containing the (null separated) names of events and states as they are defined in the machine. Then, whenever you want to monitor an individual machine instance's state just add a trace() call to your sketch's setup() method.

```c++
Atm_blink led;

void setup() {
  Serial.begin( 9600 );
  led.trace( Serial );
  led.begin( 4, 250 );
  led.trigger( led.EVT_ON );
}
```

Now, when you run the tutorial example with the Serial monitor your sketch will produce output similar to this:

```
0 Switch BLINK@52E from *NONE* to IDLE on *NONE* (1 cycles in 0 ms)
1 Switch BLINK@52E from IDLE to LED_ON on EVT_ON (2 cycles in 0 ms)
265 Switch BLINK@52E from LED_ON to LED_OFF on EVT_TIMER (4204 cycles in 200 ms)
477 Switch BLINK@52E from LED_OFF to LED_ON on EVT_TIMER (4198 cycles in 200 ms)
689 Switch BLINK@52E from LED_ON to LED_OFF on EVT_TIMER (4198 cycles in 200 ms)
901 Switch BLINK@52E from LED_OFF to LED_ON on EVT_TIMER (4198 cycles in 200 ms)
1113 Switch BLINK@52E from LED_ON to LED_OFF on EVT_TIMER (4198 cycles in 200 ms)
1326 Switch BLINK@52E from LED_OFF to LED_ON on EVT_TIMER (4197 cycles in 200 ms)
```

You can monitor exactly what machine switched from one state to another, on which event trigger and the time (in milliseconds) at which it occurred. You can monitor many machines at once in this manner. You can distinguish the machines by their class name and the machine object's hex address.

```c++
Atm_blink led1, led2;

void setup() {
  Serial.begin( 9600 );
  led1.trace( Serial );
  led2.trace( Serial );
  led1.begin( 4, 250 ).trigger( led1.EVT_ON );
  led2.begin( 5, 250 ).trigger( led2.EVT_ON );
}
```

### Wrap up ###

The advantage of using state machines is that you can run dozens of state machines at the same time, each performing its own subtask. The appliance scheduler object keeps them all running at their selected priority. The machines can communicate asynchronously via the message queue and you can easily perform complicated tasks while your Arduino stays responsive to user input. Many of today's hottest technologies like Node.JS and Python's twisted are based on event driven frameworks. Now your Arduino can have one too.

For the sake of this tutorial I've kept everything in a single .ino file, but ideally a state machine would be packaged in its own separate .cpp and .h files that can be distributed and shared just like any Arduino library. Look at the source of one of the bundled state machines to see how that's done. 

The blink example included in the library contains the tutorial source code, the blink_modular example has the same code with the blink machine in separate .cpp/.h files. For a more capable led blinker state machine have a look at the Atm_led and Atm_fade state machines.

