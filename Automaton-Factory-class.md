Runs state machines according to their priority.

### add( machine ) ###

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

### cycle() ###

Executes a factory cycle. In a factory cycle all priority queues are cycled a number of times corresponding to their priority level. Called from the Arduino loop().

```c++
void loop() {
	factory.cycle();
}
```

### find( label ) ###

Finds a machine that has previously been added to the factory according to its label. Returns a pointer to the machine object.

```c++
Machine * led;
led = factory.find( 'LED_R' );
led->msgWrite( 0 );
```
