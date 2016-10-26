
# DigitalScope - Library

This library is a collection of routines for recording events from digital input pins. Turns
your Arduino into a software logic analyzer. Carefully designed maximizing capture performance.

## Usage

Include the library and configure a new scope.

```c++
#include <DigitalScope.h>
using namespace cheind;

// Initalize scope with max number of events to collect (128) and target pin (2)
#define NEVENTS 128
DigitalScope<NEVENTS> scope(2);
```

**DigitalScope** uses interrupt service routines to free your code from unnecessary polling.
One way of using a scope is by recording events during a fixed time period after some start
trigger was observed.

```c++

// Will be used to signal begin of event detection. 
bool started = false;

void setup()
{
    // Used for outputting captured values.
    Serial.begin(9600);
    while(!Serial) {}

    // Set a callback function to be invoked when data recording has begun. 
    scope.setBeginCallback(onBegin);

    started = false;

    // Start recording when we observe a falling edge on input.
    scope.start(FALLING);
}
```

The callback function simply set the `started` flag as shown below

```c++
void onBegin() 
{
    // Signal loop() that first event was recorded.
    started = true;
}
```

Finally in `loop()` we wait for 1 second once `started` turned true and 
output all events captured during this period via the Serial interface.

```c++
void loop()
{
    if (started) {

        // Sleep for a second. 
        // Scope will continue to record data meanwhile.
        delay(1000);
        
        // Stop recording
        scope.stop();

        Serial.println("BEGIN DATA");

        const uint16_t nEvents = scope.numEvents();        
        for (uint16_t i = 0; i < nEvents; ++i)
        {
            // Print event time in microseconds since first event, 
            // event type RISING/FALLING and
            // resulting event state HIGH/LOW

            Serial.print(scope.timeOf(i)); Serial.print(" ");            
            Serial.print(scope.eventOf(i)); Serial.print(" ");
            Serial.print(scope.stateOf(i));

            Serial.println();
        }

        Serial.println("END DATA");

        // Restart the scope after a short pause.

        delay(5000);        
        started = false;
        scope.start(FALLING);
        Serial.println("LOG Ready for capture");
    }
}
```

More examples can be found in the [examples/](examples/) directory.

 
 