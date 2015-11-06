This state machine monitors an analog input with a configurable sample period and fires off a callback whenever one of a list of thresholds are crossed. Optionally keeps a moving average to smooth out peaks and troughs.

* [begin()](#atm_comparator--begin-int-attached_pin-int-samplerate-triggercb_t-cb-)
* [threshold()](#atm_comparator--threshold-uint16_t--v-uint16_t-size--bool-catchUp-false-))
* [average()](#atm_comparator---average-uint16_t--v-uint16_t-size-)
* [onSwitch()](#machine--onswitch-swcb_sym_t-callback-const-char-sym_s-const-char-sym_e-)

## Synopsis ##

```c++
#include <Automaton.h>
#include <Atm_comparator.h>

Atm_comparator cmp;
Factory factory;

void cmp_callback( int v, int up, int idx_threshold, int v_threshold )
{
  // Do something when one of the thresholds is crossed
}

void setup()
{
  static uint16_t threshold_list[] = 
    { 100, 200, 300, 400, 500, 600, 700, 800, 900, 1000 }; 

  cmp.begin( A2, 50, cmp_callback );
  cmp.threshold( threshold_list, 10 );
  factory.add( cmp );
}

void loop()
{
  factory.cycle();
}
```

### Atm_comparator & begin( int attached_pin, int samplerate, triggercb_t cb ) ###

Attaches the comparator to an analog input pin, sets the sample period and the callback routine. The sample period is in milliseconds per sample.

```c++
void cmp_callback( int v, int up, int idx_threshold, int v_threshold )
{
  // Do something when one of the thresholds is crossed
}

void setup()
{
  ...
  cmp.begin( A2, 50, cmp_callback );
  ...
}
```

The callback has 4 arguments:

Argument | Function
-------- | --------
v | the last measured value (or moving average)
up  | The direction in which the threshold was crossed (1 = up, 0 = down)
idx_threshold | The index of the threshold that was crossed
v_threshold | The value of the threshold that was crossed

### Atm_comparator & threshold( uint16_t * v, uint16_t size, bool catchUp=false ) ###

Sets a list of thresholds to monitor. If the measured analog input crosses one of these thresholds the callback is called. Arguments are a pointer to a list of 16 bit unsigned integers and the size of the list.

```c++
void setup()
{
  static uint16_t threshold_list[] = 
    { 100, 200, 300, 400, 500, 600, 700, 800, 900, 1000 }; 

  cmp.begin( A2, 50, cmp_callback );
  cmp.threshold( threshold_list, 10 );
}
```
Declare the list as a global variable or as *static*. The threshold list can hold a maximum of 64 entries.

If the optional catchUp argument is set to `true`
```c++
cmp.threshold( threshold_list, 10, true );
```
the state machine will catch up to the current value as soon as the state machine starts cycling. It does this by calling your callback as many times as necessary if the current value is larger than at least one threshold value. So in the above situation, if you have a thermistor attached to A2 and its current value is 511, the callback will be called 5 times in quick succession, once each for the 100, 200, 300, 400 and 500 threshold values. If catchUp is set to `false` or omitted from the `threshold()` call, when the state machine starts, the callback will not be called until a threshold is crossed.

### Atm_comparator &  average( uint16_t * v, uint16_t size ) ###

Connects an averaging buffer to the state machine. This will cause the state machine to monitor a *moving average* instead of the momentary value. Tune the size of this buffer and the sample rate to get the smoothing behavior you want.

```c++
uint16_t avgbuffer[256];

void setup()
{
  static uint16_t threshold_list[] = 
    { 100, 200, 300, 400, 500, 600, 700, 800, 900, 1000 }; 

  cmp.begin( A2, 50, cmp_callback );
  cmp.threshold( threshold_list, 10 );
  cmp.average( avgbuffer, 256 );
  factory.add( cmp );
}
```
The buffer variable is used as a ring buffer to store the sampled values. The value the comparator uses to check the thresholds is computed as the average of the values in the ring buffer. The call to average() fills up the ringbuffer with samples so the average will make sense right from the start.

### Machine & onSwitch( swcb_sym_t callback, const char sym_s[], const char sym_e[] ) ###

To monitor the behavior of this machine you may connect a monitoring function with the Machine::onSwitch() method. 

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
```

Use the code below to pass STATES and EVENTS symbol tables to the state machine, open up a serial terminal and watch the machine change states. 

```c++
cmd.onSwitch( sw, "IDLE\0SAMPLE\0SEND","EVT_TRIGGER\0EVT_TIMER\0ELSE" );
```

**WARNING: This machine changes state for every sample taken and will produce a lot of log output quickly**
