Download the library from Github  by clicking here: 

<https://github.com/tinkerspy/Automaton/archive/master.zip>

Save the zip file Automaton-master.zip to your desktop. Uncompress the zip file, you should end up with a folder called 'Automaton-master' which contains 'Automaton.cpp' and other files. 

(if the folder contains another folder called 'Automaton-master' you should use that one instead)

Rename the folder from 'Automaton-master' to 'Automaton'.

To install the library, first quit the Arduino application. Drag the Automaton folder into your Arduino libraries folder. Under Windows, it will likely be called "My Documents\Arduino\libraries". For Mac users, it will likely be called "Documents/Arduino/libraries". On Linux, it will be the "libraries" folder in your sketchbook.

Your Arduino library folder should now look like this (on Windows):

- My Documents\Arduino\libraries\Automaton\examples
- My Documents\Arduino\libraries\Automaton\extras
- My Documents\Arduino\libraries\Automaton\src
- ....


or like this (on Mac and Linux):

- Documents/Arduino/libraries/Automaton/examples
- Documents/Arduino/libraries/Automaton/extras
- Documents/Arduino/libraries/Automaton/src
- ....

Start your Arduino application and the library should be under `Sketch > Include library`.


## Quick start ##

Connect an led and an appropriate current limiting resistor (330 ohm will probably do) between pin 4 and GND.

In the Arduino menu choose: `File > Examples > Automaton > blink_modular`

Run the example sketch.

```c++

#include <Automaton.h>
#include "Atm_blink.h"

Atm_blink led;

void setup() {
  led.begin( 4, 200 );        // Setup a blink machine
  led.trigger( led.EVT_ON );  // Turn it on
}

void loop() {
  led.cycle();
}
```

If your led blinks the installation was successful!
