The idea is to subdivide your program into subtasks, usually but not exclusively, centered around one or more inputs or outputs end to create a Finite State Machine (or Automaton) for each of them. The machines can interact with each other by setting the other machines' states or sending signals or messages. In my experience this results in easy to understand code that runs extremely stable. In fact most of your application won't be coded, but described as a table of states that uses pins, timers and counters as input. 

Traditionally, let's start with the simplest possible application: blinking a Led.

### Subclass Automaton ###

To create a new Automaton (state machine) you must first subclass the Automaton base class. We also give it a short class label by which we can recognize it later, when it's up and running.

```c++
#include <Automaton.h>

class Blink : public Machine {

  public:
    Blink( void ) : Machine() { class_label = "BLNK"; };

}

void setup()
{
}


void loop()
{
}
```

### The beginning ###

The Blink subclass needs a new constructor that sets the class_label and calls the parent class' constructor
, you want to be able to pass it two arguments, the pin that the led is connected to and the blinkrate in milliseconds. You also need the state table that will contain the Led class's possible states, say LED_ON and LED_OFF.

You need a pin variable to hold the number of the pin the Led object is attached to and when you're at it you can set the pinmode to OUTPUT here. We'll also be needing a millisecond timer, so we declare that as well. We'll aslo need an *EVENTS* enum which for now we fill with just the *ELSE* pseudo-event which must always be the last of the EVENTS.

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
}
```

### The State table ###

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

For now the state table consists of two rows (the LED_OFF and LED_ON states we just thought of) and four columns. Al fields are filled with the value -1, which means as much as 'do nothing'. And that's exactly what our new machine does - for now.

