Multitasks Automaton state machines. 

<!-- md-tocify-begin -->
* [component()](#appliance--component-machine--machine-)  
* [run()](#appliance--run-uint32_t-time--0-)  

<!-- md-tocify-end -->

### Appliance & component( Machine & machine ) ###

Adds a State Machine (component) to the app. 

```c++
void setup() {
  app.component( led1.begin( 5 ) );
}

void loop() {
  app.run();
}
```	

### Appliance & run( uint32_t time = 0 ) ###

Executes a appliance cycle. In a appliance cycle all machines are cycled once. Normally called from the Arduino loop().

```c++
void loop() {
  app.run();
}
```

When the *time* argument is specified and greater than zero, the run() method will cycle until the corresponding number of milliseconds has passed. This can be useful if you want to run one or more state machines in a sequential pattern (normally from the setup method).

```c++
void setup() {
  app.component( led1.begin( 5 ).blink( 1000, 1000 ) ); // Blink a led slowly
  app.component( led2.begin( 6 ).blink(  100,  100 ) ); // Blink a led quickly
  led1.trigger( led1.EVT_BLINK ); // Start them both blinking
  led2.trigger( led2.EVT_BLINK );
  app.run( 10000 ); // Let them blink for 10 seconds
  led1.trigger( led1.EVT_OFF ); // Stop the blinking
  led2.trigger( led2.EVT_OFF );
}
```


