Multitasks Automaton state machines according to their priority. This is a great way of running an application that consists entirely of state machines. If you want to mix state machines with other code in the Arduino loop(), you may be better off calling the individual machine's cycle() methods instead.  

* [add()](#factory--add-machine--machine-)
* [cycle()](#factory--cycle-void-)
* [find()](#machine--find-const-char-label-) *

(methods marked with an asterisk * are not available in a TinyFactory)

### Factory & add( Machine & machine ) ###

Adds a Machine object to the factory. Places it into the appropriate queue according to its priority setting. (See the Machine::priority() method documentation for more info)

```c++
void setup() {
	led1.begin( 4 );
	led1.priority( 2 );
	factory.add( led1 );
}

void loop() {
	factory.cycle();
}
```	
After adding a machine to the factory, the add() method will cycle() it exactly once to make sure it's in an initialized state.

### Factory & cycle( void ) ###

Executes a factory cycle. In a factory cycle all priority queues are cycled a number of times corresponding to their priority level. Normally called from the Arduino loop().

```c++
void loop() {
	factory.cycle();
}
```

### Machine * find( const char label[] ) ###

Finds a machine that has previously been added to the factory according to its label. Returns a pointer to the machine object.

```c++
Machine * led;
led = factory.find( 'LED_R' );
led->msgWrite( 0 );
```

### Machine * msgSend( const char label[], int msg ) ###

Sends a message to the machine(s) with instance label 'label'. Returns number of matching machines found.

```c++
Machine * led1, led2, led3;

led1.begin( 3 ).label( "LED_R" );
led1.begin( 4 ).label( "LED_G" );
led1.begin( 5 ).label( "LED_B" );

factory.add( led1 ).add( led2 ).add( led3 );

factory.msgSend( "LED_R", Atm_led::MSG_BLINK ); // Blinks red led
```


### Machine * msgSendClass( const char label[], int msg ) ###

Sends a message to the machine(s) with class label 'label'. Returns number of matching machines found.

```c++
Machine * led1, led2, led3;

led1.begin( 3 ).label( "LED_R" );
led1.begin( 4 ).label( "LED_G" );
led1.begin( 5 ).label( "LED_B" );

factory.add( led1 ).add( led2 ).add( led3 );

factory.msgSendClass( "LED", Atm_led::MSG_BLINK ); // Blinks all leds
```
