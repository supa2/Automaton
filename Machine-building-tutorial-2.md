This tutorial demonstrates the process of building a custom Automaton component (state machine) using the [Machine Editor](http://www.wolkendek.nl/atm/). The Machine Editor is a tool for creating a state machine template. I takes care of most of the grunt work involved in building a machine. The Editor produces templates that normally just require some editing to customize the begin(), event() and action() methods.

We use an object most of us encounter every day as a subject, a traffic light. Our goal is to create a 
traffic light state machine that can be controlled with commands (or events) so it will easily integrate
with the other Automaton state machines. We also want an automatic mode so that the light will be able 
to cycle through its phases on its own. We use the Dutch traffic light sequence: Green -> Yellow -> Red -> Green.

![Machine Editor](images/mb2a.png)

To start building the state machine template go to [http://wolkendek.nl/atmdev/index.php](http://wolkendek.nl/atmdev/index.php) and press the *Create new blank state machine* button. Then think of a good name for your machine 
(I suggest Atm_trafficlight) and press *Rename state machine*. The Machine editor is now ready to start defining 
your machine. Click the *States* option in the top menu to enter the state table editor.

![State Table Editor](images/mb2b.png)


