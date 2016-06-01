Chapter 14: Putting C to Work with Pebble Smartwatch Sensors
=======
We have a considerable number of chapters learning and practicing C programming.  We have a few more things to discuss, but this seems like a good time to pause and review how we can use C effectively on a Pebble Smartwatch.  Through the Project Exercises, we have seen many applications that use drawing, color, timers and other system tools to do many applications.  In this chapter, we are going to put our knowledge of C to work in an area we have not explored: smartwatch sensors.  

As we have seen back in Chapter 1, there are several sensors packed into a smartwatch.  Each watch has a 3-axis accelerometer, a magnetometer, and an ambient light sensor.  The Pebble Time smartwatch series also has a microphone built in, which might not technically be a sensor, but we'll count so we can examine how to program it.  We will also access the battery information, but it too is technically not a sensor. Accessing the timer is also not a sensor, but we will discuss it anyway.  We will spend time in this chapter reviewing each of these sensors and the structures provided by Pebble to access these sensors using C.

As an added bonus, although it is not a sensor, we will also consider how to work with the vibrating motor.

> ** An LED on the Pebble Steel: Another Bonus? **
> 
While this does not fall into the sensor category either, the Pebble Steel features a multi-coloured LED on the bezel to indicate charging status. When the Pebble Steel was released, this accessory came with no programming interface.  While the operating system could work with it, programmers could not access this LED.  
>
This unique light did not make into the design for the smartwatch models that followed it.  At the present time, there is still no programming API to access this LED.

### Some General Notes ###

Let's make a few notes that apply across all sensor access.  

First, it's important to remember that the contents of this chapter are an *application* of C, not part of the C programming language.  Discussing these topics is very instructive, because it serves as a great example of the information we have covered.  It's good practice to work with these features of smartwatches.  But the information provided here is specific to Pebble smartwatches and not available anywhere else.

Secondly, accessing the data provided by these services and sensors provides a great example of feature access in general.  This kind of access and the data it provides are usually provided in their own struct declaration, dynamically allocated, accessed through pointers.  All the practice we have done up until now will be very useful, since these dynamic structures will require careful allocation and deallocation, as well as careful access.

Finally, the programming interfaces we will discuss for the access detailed here are provided by Pebble and are subject to change.  We will try to keep up with any changes in this book, but new version of the SDK might slip in changes before we add them here.

### Programming the Accelerometer ###

The accelerometer on a Pebble smartwatch records movement data in 3 dimensions.  Really, the data collection contains *acceleration* data in 3 dimenensions.  If you could start a smartwatch moving in a direction, then continuing moving at exactly the same speed, no data would be collected. But this is not (typically) possible, so collecting acceleration data is quite effective.  

Acceleration data comes in two forms: raw data and processed data. Raw acceleration data is a simple sample of accelerometer data, given in three dimensions.  Processed acceleration data includes raw data, a timestamp of when the sample of the data was made and an interpretation of whether the smartwatch vibrated when the sample data was collected.

In addition to acceleration data, a Pebble smartwatch also registers *tap* data.  Tap data is an abstraction of accelerometer data to form a *tap event*.  A certain pattern of acceleration data can be translated to a tap event, and since this is useful data to know, tap events are available in addition to acceleration data. 

When sampling accelerometer data, we can sample in two ways.  We can sample *manually*, calling a function to get data whenver that data is needed.  The other way is to *subscribe to events* in much the say we have seen before.  With subscriptions, a callback function is called when an event is detected or data is sampled.  

#### Raw Accelerometer Data ####

Let's consider the raw data structure `AccelRawData`: 

    typedef struct __attribute__((__packed__)) {
      int16_t x;
      int16_t y;
      int16_t z;
    } AccelRawData;

Here, we see that raw acceleration data is an *(x, y, z)* triple of acceleration data in 16-bit integers.  Note that we have the GNU C compiler attribute specifier `__attribute__` specifying the `__packed__` attribute for the struct.  Remember this from Chapter 13: there is no padding to be inserted anywhere in this struct; the data is stored back-to-back.  

Raw accelerometer data can only be obtained through event subscription.  The function to subscribe to raw sample events has the following header:

    void accel_raw_data_service_subscribe(uint32_t samples_per_update, AccelRawDataHandler handler) 
    
The parameters are the number of samples to save in a buffer before calling the event handler (`samples_per_update`) and the name of the event handler itself (`handler`).  The sample event handler must have the following handler:

    void raw_data_handler(AccelRawData *data, uint32_t num_samples, uint64_t timestamp)
    
The parameters sent to this function are the latest data sample (in `data`), the number of samples available since that data event (`num_samples`), and the time the last sample occured (`timestamp`).  Note that if the maximum number of samples waiting to be analyzed is 25, so some data might be lost if more than 25 samples were taken between data events.

Let's take an example.  Consider a simple application where accelerometer data is collected and displayed to an application log.  We start in the `init` code for the app:

      accel_raw_data_service_subscribe(ACCEL_SAMPLING_100HZ, raw_data_handler);

Note the `ACCEL_SAMPLING_100HZ`; this is part of an `AccelSamplingRate` enum and describes sampling at 100 times per second.  By using a simple data handler such as 

    void raw_data_handler(AccelRawData *data, uint32_t num_samples, uint64_t timestamp) {
        APP_LOG(APP_LOG_LEVEL_INFO, "In Raw Data Handler, samples = %u, time = %lu", 
                (unsigned int)num_samples, (long unsigned int)timestamp);
        APP_LOG(APP_LOG_LEVEL_INFO, "X = %d, Y = %d, and Z = %d", data->x, data->y, data->z);
    }
    
We get the following automatic output:  

    [INFO] button_click.c:8: In Raw Data Handler, samples = 25, time = 3828525699
    [INFO] button_click.c:9: X = 0, Y = 0, and Z = -1000
    [INFO] button_click.c:8: In Raw Data Handler, samples = 25, time = 3828526774
    [INFO] button_click.c:9: X = 0, Y = 0, and Z = -1000
    [INFO] button_click.c:8: In Raw Data Handler, samples = 25, time = 3828527774
    [INFO] button_click.c:9: X = 0, Y = 0, and Z = -1000

and more printing every second.  This is simulated data in the CloudPebble emulator.  If we run this on a real watch, we get the following output:

OUTPUT

Note also that the timesatamp is in "Unix epoch format", that is, seconds since 12:00 am on January 1, 1970.

#### Processed Accelerometer Data ####

The processed accelerometer data has the following format:

    typedef struct __attribute__((__packed__)) AccelData {
      int16_t x;
      int16_t y;
      int16_t z;
      bool did_vibrate;
      uint64_t timestamp;
    } AccelData;

Here, we can see that an indication of vibration and a timestamp have been added to the raw accelerometer data we saw in the previous section.  

Processed data is available manually or by subscription.  To get a data sample manually, we use the `accel_service_peek` function call and make sure that we do not subscribe to sampling events.  For example, we can connect a manual data sample collection to occur when "select" button is pressed.  The "select" button click handler would look like the following:

    static void select_click_handler(ClickRecognizerRef recognizer, void *context) {
       AccelData *data = (AccelData *)malloc(sizeof(AccelData));
       int err = accel_service_peek(data);
    
       APP_LOG(APP_LOG_LEVEL_INFO, "In the selectclick handler, time = %lu", 
                                   (long unsigned int)(data->timestamp));
       APP_LOG(APP_LOG_LEVEL_INFO, "X = %d, Y = %d, and Z = %d", data->x, data->y, data->z);
   }
   
Note here that we sent the `accel_service_peek` function an `AccelData` object that was already allocated.  The function filled in the sturcture and returned an indication of error.  (An integer value is returned: 0 if no error occured, -1 if an error occured, -2 if a previous subscription is in place.)
    
Subscriptions are handled in much the same way they are done with raw data samples.  Subscription to the event service is done through `accel_data_service_subscribe`, the header for which is shown below:

    void accel_data_service_subscribe(uint32_t samples_per_update, AccelDataHandler handler)
    
When you subscribe to the accelerometer service, you must give the number of samples in each event update (`samples_per_update`) and a function that will be called when that many samples have been collected.  The handler looks a lot like the handler for raw data:

    void processed_data_handler(AccelData *data, uint32_t num_samples)
    
Here, the handler would be called with the latest data (in `data`) and the number of samples in the data queue (`num_samples`).  Note that the information *not* included in the raw data structure are included here, but they are just the data that included as parameters to the raw data handler.

#### Tap Events ####

Tap events are a combination of accelerometer data samplings that, taken  together, can be interpreted as a tap.  "Tap" is not the most accurate description of the event; "shake" or "flick" is the best description here.  Taps will likely *not* be recorded because the cause very little movement of the smartwatch.

Because there are several data samples combined for a tap, there is no "raw" data for a tap and there is no manual tap sampling.  The only way to get taps is to register a callback to be called when taps happen.

To register for tap events, you need to call `accel_tap_service_subscribe`, whose header looks like this:

    void accel_tap_service_subscribe(AccelTapHandler handler)
    
An `AccelTapHandler` is a function whose header look like that below:

    void tap_handler(AccelAxisType axis, int32_t direction)
    
Here, we get some interesting information.  The `axis` parameter will depict what axis the tap occured on; this is an enum value, one of `ACCEL_AXIS_X`, `ACCEL_AXIS_Y`, or `ACCEL_AXIS_Z`.   The `direction` parameter describe which direction along the axis the tap occured; it's value is either `1` or `-1` for positive or negative (respectively) movement along the tap. 

Let's take an example.  Suppose we simply want to be notified if a tap event has occured.  We can use a tap handler like this:

    void tap_handler(AccelAxisType axis, int32_t direction){
        APP_LOG(APP_LOG_LEVEL_INFO, "Tap! along axis %s, direction = %d", 
                (axis == ACCEL_AXIS_X ? "X" :
                   (axis == ACCEL_AXIS_Y ? "Y" : "Z")), (int)direction);
    }
   
and we subscribe to the tap event service with this call in the `init` function of our app:

    accel_tap_service_subscribe(tap_handler);

Now flicks of your wrist will produce output like that below:

    [INFO] button_click.c:10: Tap! along axis Y, direction = -1
    [INFO] button_click.c:10: Tap! along axis Z, direction = 1
    [INFO] button_click.c:10: Tap! along axis Z, direction = -1
    [INFO] button_click.c:10: Tap! along axis Z, direction = 1
    [INFO] button_click.c:10: Tap! along axis X, direction = -1
    [INFO] button_click.c:10: Tap! along axis Y, direction = 1

### Accessing Magnetometer and Compass Data ###

A magnetometer is an instrument that measures the direction and strength of a magnetic field.  In a Pebble smartwatch, a magnetometer can be used to calculate the smartwatch's position relative to the Earth's magnetic north.  The operating system combines magnetometer measurements with accelerometer data to both calibrate a compass and to provide data on the heading of smartwatch with respect to magnetic north.  

As with the accelerometer, access to the magnetometer data can be manual or based on subscription.  Manual access to this data is done using `compass_service_peek` with this header:

    int compass_service_peek(CompassHeadingData *data)

TThe `CompassHeadingData` is a struct:

    typedef struct {
      CompassHeading magnetic_heading;
      CompassHeading true_heading;
      CompassStatus compass_status;
      bool is_declination_valid;
    } CompassHeadingData;

`CompassHeading` is a 32-bit integer and describes the angle from the current orientation of the smartwatch to magnetic north.  `CompassStatus` is an enum that describes the current state of compas calibration: calibrating with invalid data (`CompassStatusDatatInvalid`), calibrating with valid data (`CompassStatusCalibrating`), and calibration completed (`CompassStatusCalibrated`).  The struct field `is_declination_valid` is not used in the current version of the SDK. 

The operating system also provides a compass subscription service that makes updates as to directional heading.  To get updated on compass heading, you must subscribe using `compass_service_subscribe`, whcih has this header:

    void compass_service_subscribe(CompassHeadingHandler handler)
    
The `CompassHeadingHandler` is a function that has the following header:

    void compass_heading_handler(CompassHeadingData heading)

As an example, let's use a simple heading handler that gives the direction we are heading.  We could write a handler that looks like this:

    void heading_handler(CompassHeadingData heading) {
        APP_LOG(APP_LOG_LEVEL_INFO, "Compass heading is %d degrees from north.", 
                                    (int)heading.magnetic_heading); 
    }

and we register with the compass service this way:

    compass_service_subscribe(heading_handler);
    
So let's take an example.  Let's say that we use this code as the handler for the select button:

    static void select_click_handler(ClickRecognizerRef recognizer, void *context) {
        CompassHeadingData data;
        int compass = compass_service_peek(&data);
    
        APP_LOG(APP_LOG_LEVEL_INFO, "In the selectclick handler, CompassHeading = %d, status = %d", 
            (int)data.magnetic_heading, (int)data.compass_status);
    }

We want to see the values for the magnetic heading and for the status of the compass.  We would expect the compass status to be `CompassStatusCalibrated` and we should get some valid data we can use as a directional heading. We get several lines of output that look like this:

    [INFO] button_click.c:52: In the selectclick handler, CompassHeading = 58556, status = 0 
    
This is an odd value for the heading and `0` is not the value we expected for the status.  First, the `0` value says that the sensor is calibrating and we need to be patient and wait for it.   Secondly, the compass heading value must be adjusted and compared to magnetic north.  Fortunately, there's a macro define for this.  We need to use

    TRIGANGLE_TO_DEG(data.macgnetic_north)
    
When we use this converter, east becomes approximately 309 degrees, which makes more sense.  

### Using the Ambient Light Sensor ###

Pebble smartwatches have a complex and thorough health data service built into their operating system.  As part of this health data, the ambient light level around a smartwatch is recorded.  Light levels are available to programmers as one of 5 different values, available as an enum:

    typedef enum AmbientLightLevel {
      AmbientLightLevelUnknown = 0,
      AmbientLightLevelVeryDark,
      AmbientLightLevelDark,
      AmbientLightLevelLight,
      AmbientLightLevelVeryLight,
    } AmbientLightLevel;

To obtain the light level, we must query the health system. Querying the health system gives us a struct, `HealthMinuteData`,  shown below:

    typedef struct {
      uint8_t steps;  
      uint8_t orientation;  
      uint16_t vmc; 
      bool is_invalid: 1;   
      AmbientLightLevel light: 3; 
      uint8_t padding: 4;
      uint8_t reserved[7];   
    } HealthMinuteData;
    
For ambient light levels, we are interested (naturally) in the `light` field.  A call to the function `health_service_get_minute_history` will get us a value for this field.  The header for this function is below:

    uint32_t health_service_get_minute_history(HealthMinuteData *minute_data, 
                                               uint32_t max_records,
                                               time_t *time_start, time_t *time_end)
                                               
This function fills in `minute_data`.  This collection is an *array* of `HealthMinuteData` structs.  The number in this array is given by `max_records` and the time to check starts at `time_start` and ends at `time_end`.  

There are subscription-based access methods that give health data.  However, none give `HealthMinuteData` and they are based on health data parameters, like steps taken or sleep time, and not necessarily on the ambient light.  So manual acquisition of light data is the only reliable access method.

So, let's say we want to write a function to return the ambient light level.  Such a function is going to have to call the above function and extract the light level.  

> ** When Minute Data is Not Really Minute Data **
> 
Despite the fact that the data structure is labelled `HealthMinuteData`, the health data is not gathered every minute.  The underlying health data is actually updated every 15 minutes, meaning the most recent `HealthMinuteData` value returned may be up to 14 minutes old.  So there really is no realtime source of the ambient light level.

### Accessing Battery Information ###

The Pebble smartwatch battery level is available much like other sensor information is available: *manually* and *through subscription*.  

Battery charge information is revealed in a struct, as below:

    typedef struct {
      uint8_t charge_percent;
      bool is_charging;
      bool is_plugged;
    } BatteryChargeState;

The percentage charged state of the battery is given, along with information about charging and whether the watch is plugged in.  

Manual retrieval of battery inforamtion is done through the `battery_state_service_peek` function, whose header is below:

    BatteryChargeState battery_state_service_peek(void)
    
It needs no parameters and returns a `BatteryChargeState` struct.

Subscriptions follow the pattern we have seen.  There is a function to register a callback function and to subscribe to a charge service; there is a function to unsubscribe from the service.  The call back function's header is

    void battery_state_handler(BatteryChargeState charge)
    
So, let's say we want to check the battery charge state when we press the "select" smartwatch button.  Here's a version of `select_click_handler` that would do this:

    static void select_click_handler(ClickRecognizerRef recognizer, void *context) {
        BatteryChargeState charge = battery_state_service_peek();
    
        APP_LOG(APP_LOG_LEVEL_INFO, "In the selectclick handler, battery level = %d", charge.charge_percent);
        APP_LOG(APP_LOG_LEVEL_INFO, "pebble is %s plugged in and is %s charging.", 
                                    (charge.is_plugged?"":"not"), (charge.is_charging?"":"not"));
    }
    
This gives the following output:

    [INFO] button_click.c:50: In the selectclick handler, battery level = 40
    [INFO] button_click.c:52: pebble is not plugged in and is not charging.

### Using Timers ###

In a smartwarch, timers are an essential concept to implement.  In Pebble smartwatches, timers have a rich implementation.

There are actually *two* types of timers used by Pebble smartwatches: *tick* timers and *sleep* timers.  These timers are similar in that they both call a callback function when the timer expires.  The difference between them is tick timers *automatically* renew and call the callback function in specific intervals while sleep timers only fire once, calling their callback function, and need to be renewed explicitly in the program code.

To use a tick timer, we need a tick timer callback function, described by the header below:

    void tick_handler(struct tm *tick_time, TimeUnits units_changed)
    
Here, the `struct tm` structure is a standard way to reference time, and looks like:

    struct tm {
        int tm_sec;         /* seconds */
        int tm_min;         /* minutes */
        int tm_hour;        /* hours */
        int tm_mday;        /* day of the month */
        int tm_mon;         /* month */
        int tm_year;        /* year */
        int tm_wday;        /* day of the week */
        int tm_yday;        /* day in the year */
        int tm_isdst;       /* daylight saving time */
    };
    
The `TimeUnits` is an enum that contains information about what time unit changed from the last call to this one:

    typedef enum {
      SECOND_UNIT = 1 << 0,
      MINUTE_UNIT = 1 << 1,
      HOUR_UNIT = 1 << 2,
      DAY_UNIT = 1 << 3,
      MONTH_UNIT = 1 << 4,
      YEAR_UNIT = 1 << 5
    } TimeUnits;

This enum is interesting because there could be several different units represented in the same "bitmask".  For example, if the `MINIUTE_UNIT` changed *and* the `HOUR_UNIT` changed, you could represent them both as 

    MINUTE_UNIT | HOUR_UNIT
    
because they are each set up to be represented by a unique bit position.  

Sleep timers work as expected: the callback registered by the call to `app_timer_register` will be called when the timer expires.  This function has the header:

    AppTimer *app_timer_register(uint32_t timeout_ms, AppTimerCallback callback, void *callback_data)
    
The `timeout_ms` parameters specifies the amount of time, in milliseconds, until the timer exprires.  The `callback` specified a callback function, whose header must be

    void timer_callback(void *data)
    
Note that the `data` parameter here is not specified; it is given by the `callback_data` parameter in the `app_timer_register` call.
    
We have seen sleep timer calls in previous chapters.  For example, in Chapter 10, in the snake game for Project 10.2, we used the tick timer to power the snake.  Every time the tick timer expired, we moved the snake over a square.  For example, the time was initialized with this call:

    app_timer_register(TICK_TIME_MS, refresh_timer_callback, NULL);
    
The `TICK_TIME_MS` parameter had the value `400`, which made this time expire after 400 milliseconds.  The callback function `refresh_timer_callback` was called upon time expiration, with `NULL` parameters, that is, no parameters.  The `refresh_timer_callback` function looked like this:

    void refresh_timer_callback(void *data) {
	    layer_mark_dirty(window_get_root_layer(window)); // Tell the layer it needs to redraw.
    }

It was defined to mark the graphics layer as dirty/redrawable.  The drawing function for the graphics layer reset the time by calling the timer register function again.

### Bonus: Running the Vibrating Motor ###

While making a smartwatch vibrate is not exactly a sensor (it's more a user interface item), it allows for a "custom vibration pattern", which uses structs and arrays in an interesting way.  And this corresponds to the reason for this chapter.

First, there are fixed patterns of vibrations that can be initiated.  The following calls will fire off certain patterns, identifiable by their names.

    void vibes_short_pulse();
    void vibes_long_pulse();
    void vibes_double_pulse();
    
And there is the required cancellation function call, to cancel any vibration that is currently in progress:

    void vibes_cancel();
    
The custom pattern vibration call is the most interesting.  A vibration pattern is characterized by an array of integers that describe the durations of on/off specifications.  There must be at least one integer (naturally), but there can be many.  An additional integer specifies the number of "segments" in the vibration pattern.

For example, we wanted to signal S.O.S. in Morse Code.  This pattern would be three short vibrations, followed by three long vibrations, followed by three short ones again.  We could specify this as follows:

    uint32_t vibrations[] = { 100, 100, 100, 100, 100, 100,
                              300, 100, 300, 100, 300, 100,
                              100, 100, 100, 100, 100};
    VibePattern sos = {
        .durations = sos,
        .num_segments = 17
    };
    
This assumes that a short vibration is 100 milliseconds, followed by 100 milliseconds of no vibration.  Long vibrations are 300 milliseconds.  Now, when we call `vibes_enqueue_custom_pattern(sos)`, we will get the S.O.S. vibration pattern on the watch.

### Summary: Style and Practice ###

We have spent considerable time and space in this chapter discussing how to work with sensors on a Pebble smartwatch.  However, the point to this chapter was not really to understand sensor programming, although that's a good result, but to understand the techniques that system programming in C uses and to practice working with those techniques.  In particular, here's a few lessons that should stand out from this chapter.

1. **System data are gathered into collections, most often structs.**  
Whenever system data need to be gathered into one place, focused on a specific function or interface, structs are used.  There are probably several ways to store system data, but structs are the best way to collect data for a specific purpose.  This way of organizing data is used in most system programming, such as Pebble and Linux systems.

2. **Dynamic allocation of space for structs is the best way to work with system data.**  
The way system structs are used is to dynamically allocate space for them when they are needed.  System data structures can be large, sometime nesting structs within structs, and memory space is best managed dynamically, with programmers paying close attention to allocating and freeing memory as needed.

3. **System structures for the Pebble SDK system have a specific style.** 
We have discussed style and pattern of declarations before.  Pebble structures have a specific style.  Here's the battery information as an example.

    typedef struct {
      uint8_t charge_percent;
      bool is_charging;
      bool is_plugged;
    } BatteryChargeState;

   Here, a typedef is used with an unnamed struct.  When structures are declared this way, further uses of `BatteryChargeState` can be done without the use of the `struct` keyword.  This makes declarations clearer and less wordy.

4. **Sometimes, writing your own functions and structures to "rephrase" the system structures will help you access the Pebble system.** 
Abstraction is a tool we can use to make things clearer and more straight forward.  There are many data structures in a Pebble application and we can use our own designs to abstract away unused details.  An example here is the use of the ambient light sensor.  It's buried in the `HealthMinuteData` structure, but writing our own function to give the ambient light can ignore all the other health variables and focus on the `light` field.  This is a common: writing our own code to focus on the specific refinement of a data structure that we need.

4. **Practicing with system structures and style will help to make you comfortable with the large number of system structures.** 
Writing applications that access all the Pebble smartwatch subsystems can be daunting.  There are many facets to a Pebble smartwatch and many different data structures that need to be used.  Practicing with these structures will make you more comfortable with them and more confident with manipulating data and shaping that data into smartwatch functionality.  Practice with small programs like those in the project exercises in this book and you will gain confidence to take on larger applications. 

### Project Exercises ###

#### Project 14.1 #### 

This project will get you to work with accelerometer data.   Start with the starter code, available here.  This code has the hooks in that will detect / measure accelerometer changes.

Add to the code to detect the *speed* of gesture changes for the watch.  You should be able to detect the difference between fast and slow movements of the watch.  Then add vibrations for both fast and slow movements: short vibrations for fast movements and longer vibrations for slower movements.

You can find an answer to the project here.

#### Project 14.2 ####

Remember Project 10.2?  It created a snake game that used the "up" and "down" buttons to change the movement of a snake on the screen.  [You can find an answer to Project 10.2 here.](https://cloudpebble.net/ide/import/github/programming-pebble-in-c/project-10-2-answer)

Change the code to replace "up" and "down" button presses with wrist gestures. A gesture "up" will move the snake up or left and a "down" gesture will move the snake down or right.  You should be able to detect *direction* of a wrist movement.  In addition, add code to vibrate the watch when the snake turns.

You can find an answer to this project here.

#### Project 14.3 ####

One more project with the snake game.  Starting with either [Project 10.2](https://cloudpebble.net/ide/import/github/programming-pebble-in-c/project-10-2-answer) or the answer to the last project, change the direction of the snake when the wrist moves in the direct desired.  This is different than a "flick" type of gesutre; you will need magnetometer data here.  Imagine a wrist held flat, but moving in two dimensions: from north to west for a left turn and north to east for a right turn.

You can find an answer here for this project.

#### Project 14.4 ####

Project 10.3, available here, added a time indicator to an "autopilot" snake.  Take this code and add a battery indicator to the watchface.  For this project, you can choose how "granular" your indicator is: anything from 4 states to 10 states would be acceptable.  Each charge state needs something (icon, percentage, number) that indicates percentage charge in a visual manner.  Choose where to put the indicator on the screen.

You can find an answer to this project here.

**Extra Challenge:** Can you make the battery follow the snake just like the time does?
