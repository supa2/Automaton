Control a led via a digital pin. Control blink speed, pause duration and number of repeats. Also very useful for controlling other types of hardware asynchronously, like pulsing buzzers or relays. On PWM enabled pins LED brightness control is available.

![Led](images/led-small.jpg)

<!-- md-tocify-begin -->
* [begin()](#atm_led--begin-int-attached_pin-bool-activelow--false-)  
* [blink()](#atm_led--blink-int-duration-)  
* [pause()](#atm_led--pause-int-duration-)  
* [fade()](#atm_led--fade-int-fade-)  
* [brightness()](#int-brightness-int-level-)
* [range()](#atm_led--range-int-tolow-int-tohigh-bool-wrap--false-)
* [levels()](#atm_led--levels-unsigned-char-map-int-mapsize-bool-wrap--false-)
* [brighten()](#int-brighten-int-v--1-)
* [lead()](#atm_led--lead-int-ms-)  
* [repeat()](#atm_led--repeat-int-repeat-)  
* [onFinish()](#atm_led--onfinish-connector-connector-argument-)  
* [EVT_ON](#evt_on)  
* [EVT_OFF](#evt_off)  
* [EVT_BLINK](#evt_blink)  
* [EVT_START](#evt_start)  
* [EVT_TOGGLE](#evt_toggle)  
* [EVT_TOGGLE_BLINK](#evt_toggle_blink) 
* [EVT_BRUP](#evt_brup) 
* [EVT_BRDN](#evt_brdn)
* [trace()](#atm_led--trace-stream--stream-)  

<!-- md-tocify-end -->

## Synopsis ##

```c++
#include <Automaton.h>

Atm_led led1, led2;

void setup() {
  led1.begin( 4 ).blink( 40, 250 ); // Setup blinking
  led2.begin( 5 ).blink( 40, 50 ); 
  led1.trigger( led1.EVT_BLINK );   // Start blinking
  led2.trigger( led2.EVT_BLINK );
}

void loop() {
  automaton.run();
}
```

### Atm_led & begin( int attached_pin, bool activeLow = false ) ###

Attaches a digital I/O pin to the Atm_led machine. The pin will be placed in *OUTPUT* mode. Set activeLow to true if you need pull the pin low to activate your led.

This is probably the shortest Automaton sketch (that actually does anything) possible.

```c++
#include <Automaton.h>

Atm_led led;

void setup() {
  led.begin( 4 ).trigger( led.EVT_BLINK );   
}

void loop() {
}
```

If you try it you'll see that the led turns on, but it does not blink. For that to happen the led machine must be constantly cycled:

```c++
#include <Automaton.h>

Atm_led led;

void setup() {
  led.begin( 4 ).trigger( led.EVT_BLINK );   
}

void loop() {
  led.cycle();
}
```

Please note that the Atm_led machine starts up in state *IDLE*. You can turn the led on by sending a EVT_ON message or start it blinking with a EVT_BLINK message. EVT_TOGGLE toggles the led on and off, EVT_TOGGLE_BLINK toggles blinking on and off. These are the available external events:

```c++
  led1.trigger( led1.EVT_ON ); // Same as led1.on()
  led1.trigger( led1.EVT_BLINK ); // Same as led1.blink()
  led1.trigger( led1.EVT_START ); // Same as led1.start()
  led1.trigger( led1.EVT_OFF ); // Same as led1.off()
  led1.trigger( led1.EVT_TOGGLE ); // Same as led1.toggle()
  led1.trigger( led1.EVT_TOGGLE_BLINK ); // Same as led1.toggle_blink()
```
EVT_START and EVT_BLINK are equivalent.

The begin() method places the machine in the following blink configuration:

```c++
  blink( 500, 500, -1 ); // Blink forever at 1 Herz
```

### Atm_led & blink( int duration ) ###
Alternatively: Atm_fade & blink( uint32_t duration, uint32_t pause_duration, uint16_t repeat_count = ATM_COUNTER_OFF )

Sets the time that the led is fully ON during a cycle in milliseconds. The three argument version sets the time the led is fully ON, the time the led is fully OFF and the number of repeats.

```c++
void setup() {
  led1.begin( 4 );
  led1.blink( 40 ); 
  led1.blink( 40, 100, 10 ); // 40ms on, 100ms off, repeat 10 times
  ...
}
```

### Atm_led & pause( int duration ) ###

Sets the time that the led is fully OFF during a cycle in milliseconds.

```c++
void setup() {
  led1.begin( 4 );
  led1.blink( 40 );
  led1.pause( 100 );
  ...
}
```

### Atm_led & fade( int fade ) ###

This is a dummy method for interface compatibility with the Atm_fade machine. It does nothing here.

### Atm_led & lead( int ms ) ###

Sets a leading waiting period before start() or on() becomes active. Default is 0.

```c++
#include <Automaton.h>

// When the button is pressed, wait for 500 ms and then blink the led 3 times

Atm_led led;
Atm_button button;

void setup() {
  
  led.begin( 4 )
    .lead( 500 )
    .blink( 200, 200 )
    .repeat( 3 );

  button.begin( 2 )
    .onPress( led, led.EVT_START );

}

void loop() {
  automaton.run();
}
```

### int brightness( int level ) ###

Sets the brightness level (relative to either range() or map()).

```c++
  led.begin( 5 )
    .brightness( 128 ); // 50% brightness

```
_IMPORTANT_ Note that setting brightness is only supported on PWM (~) enabled pins.

Returns the current setting of brightness.

### Atm_led & range( int toLow, int toHigh, bool wrap = false ) ###

Sets the brightness range, mapped to 0..255.

```c++
  led.begin( 5 )
    .range( 0, 9, true );
  
  // Button cycles through 10 brightness levels 0..9
  button.begin( 3 )
    .onPress( led, led.EVT_BRUP ); 
```

Range uses the Arduino map() function to map the toLow/toHigh values to 0..255 (the analogWrite range). 

This means that range( 0, 9 ) corresponds to (0, 28, 56, 85, 113, 141, 170, 198, 226, 255). If you need finer control over brightness, try the levels() method instead.

The wrap argument controls what happens when the sketch attempts brightness above or below the range. 

Implicitly sets ```brightness( toHigh )```.


### Atm_led & levels( unsigned char* map, int mapSize, bool wrap = false ) ###

Defines a map of brightness levels to be used by the various brightness setting commands. 

The example below uses a long press to toggle a led on/off and a short press to cycle through the 5 predefined brightness levels.

```c++
#include <Automaton.h>

unsigned char brightness[] = { 50, 100, 150, 200, 255 };

Atm_led led;
Atm_button button;

void setup() {

  led.begin( 5 )
    .levels( brightness, sizeof( brightness ), true ); // Set levels map
    
  button.begin( 3 )
    .longPress( 2, 400 )
    .onPress( 1, led, led.EVT_BRUP ) // Short press: brightness up
    .onPress( 2, led, led.EVT_TOGGLE ); // Long press: toggle on/off
}

void loop() {
  automaton.run();
}
```

Implicitly calls ```range( 0, mapSize - 1, wrap )``` and ```brightness( mapSize - 1 )```.

### int brighten( int v = 1 ) ###

Raises or lowers the brightness with one step.

```c++
  led.brighten( 1 ); // Increases brightness
  led.brighten( -1 ); // Decreases brightness
  led.brighten(); // Increases brightness
```
Returns the current brightness value;

### Atm_led & repeat( int repeat ) ###

Sets how many times the blink pattern should repeat. Default is *ATM_COUNTER_OFF* (-1) which means it will blink indefinitely. Use *1* to blink once, etc...

```c++
void setup() {
  led1.begin( 4 ).blink( 40, 100 ).repeat( 1 ).start();
  ...
}
```

The example above gives off a single 40 millisecond pulse and then goes back to sleep (state IDLE).

### Atm_led & onFinish( {connector}, {connector-argument} ) ###

This method is used to trigger another machine when the current machine's blinking sequence has finished. This can be used to create sequences of blink patterns, but you can also trigger different types of machines in this manner.

```c++
led1.begin( 4 ).blink( 500, 500, 3 ).onFinish( led2, led.EVT_BLINK );
led2.begin( 4 ).blink( 50, 50, 10 );
led1.start();
```

The example above will blink a led slowly 3 times and then blink the same led quickly 10 times.

## Virtual methods

### virtual void initLED() ###

This internal (`protected`) method is called inside `Atm_led.begin()`. It is used to initialize the led output.
The default implementation sets the specified pin to `OUTPUT` mode and to off state (0 if activeLow is false, 1 if activeLow is true). As this method is virtual, you can create your own class that is derived from `Atm_led` to initialize other I/O hardware (e.g. I2C IO extender like MCP23017).

Example (own class named `Atm_led_mcp` that has a member variable `gpio` to access an IO extender):
```C++
void Atm_led_mcp::initLED() {
	gpio.pinMode(pin, OUTPUT);
	gpio.digitalWrite(pin, activeLow ? HIGH : LOW);
}
```

### virtual void switchOn() ###
### virtual void switchOff() ###

These internal (`protected`) methods are called inside `Atm_led.action()` method to switch on (or off, respectively) the configured LED output

The default implementation sets the specified pin to On (or off) state (activeLow considered).

As these methods are virtual, you can create your own class that is derived from `Atm_led` to switch other I/O hardware (e.g. I2C IO extender like MCP23017).

Example (own class named `Atm_led_mcp` that has a member variable `gpio` to access an IO extender):
```C++
void Atm_led_mcp::switchOn() {
	gpio.digitalWrite(pin, !activeLow);
}
```
### virtual void setBrightness(int value) ###

This internal (`protected`) method is called inside `Atm_led.begin()`. It is used to set the led brightness.

The default implementation sets the brightness value with a `analogwrite()`to the specified pin.
As this method is virtual, you can create your own class that is derived from `Atm_led` to initialize other I/O hardware (e.g. I2C IO extender like MCP23017).

Example (own class named `Atm_led_mcp` that has a member variable `gpio` to access an IO extender). The I/O extender in use it not capable of controling the brightness, so an error is shown on Serial (except the brightness value is fully on or off):
```C++
void Atm_led_mcp::setBrightness(int value) {
	if (value == toHigh) switchOn(); else if(value==toLow) switchOff(); else
	Serial.printf("ERROR: Setting brightness on GPIO expander is not possible (pin: %d)\n", pin);
}
```

## Events

### EVT_ON ###

Turns the led on.

```c++
led.on();

led.trigger( led.EVT_ON );
```

### EVT_OFF ###

Turns the led off.

```c++
led.trigger( led.EVT_OFF );
```

### EVT_BLINK ###

Starts the led blinking.

```c++
led.begin( 5 ).blink( 200 );
led.trigger( led.EVT_BLINK );
```

### EVT_START ###

Starts the led blinking. (same as EVT_BLINK)

### EVT_TOGGLE ###

Toggle the led on and off.

```c++
led.begin( 5 );
led.trigger( led.EVT_TOGGLE );
```

### EVT_TOGGLE_BLINK ###

Toggle the blinking on and off.

```c++
led.begin( 5 ).blink( 100, 900 );
led.trigger( led.EVT_TOGGLE_BLINK );
```

### EVT_BRUP ###

Increases the led brightness with one step.

```c++
led.begin( 5 ).range( 0, 10 );
led.trigger( led.EVT_BRUP );
led.trigger( led.EVT_BRUP );
```

### EVT_BRDN ###

Lowers the led brightness with one step.

```c++
led.begin( 5 ).range( 0, 10 );
led.trigger( led.EVT_BRDN );
led.trigger( led.EVT_BRDN );
```

### Atm_led & trace( Stream & stream ) ###

To monitor the behavior of this machine you may connect a monitoring function with the Machine::trace() method. 

```c++
Serial.begin( 9600 );
led.trace( Serial );
```

