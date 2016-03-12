### Tips & tricks ###

- Sending and receiving messages with data
- Machines and factories within machines
- Simulating priorities with TinyFactory

### Pitfalls ###

#### Hanging messages in the message queue ####
#### Start up race condition with machine->trigger( EVT_XXX ) ####

```c++
void setup() {
  led.begin( 3 );
  led.cycle().trigger( EVT_ON );
}

void loop() {
  led.cycle();
}


```
