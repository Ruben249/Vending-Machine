# Vending-Machine
![Circuit](https://github.com/Ruben249/Vending-Machine/assets/102288264/f5f2649f-5cb5-4014-81be-882e3dd513ed)

## Index
-[Start](#start)

-[Development](#development)

-[Problems](#problems)

-[Final](#final)


#### Start
First I decided to start designing a circuit in Tinker Card, to make sure I did not generate any short circuits and that all the components were well assembled.
Here I attach the photo, in which the joystick is missing, since I have not been able to find it in the application.
![tinkercard](https://github.com/Ruben249/Vending-Machine/assets/102288264/e6459527-0087-4790-9a6b-de8bf26ad891)

Then I mounted it on the breadboard and implemented some simple codes, to see that all the sensors were really well mounted and that they sensed correctly.
Below, I put a photo of the first circuit I assembled, since I later changed it a little to remove some cables and make everything more functional.
![1ºCircuit](https://github.com/Ruben249/Vending-Machine/assets/102288264/10c3fb46-9dab-41f4-9a36-3181d2223296)

I decided that it was best to implement 220 resistors, since they are safe and allow enough electricity to flow for the sensors to work well.
![resistors](https://github.com/Ruben249/Vending-Machine/assets/102288264/81d17d79-c596-4fef-9fd4-e36685864b1a)


#### Development
The first step to program sensors and actuators is to import the corresponding libraries and assign each sensor a Pin on the board. My distribution has been the following:
``` cpp
#include <DHT.h>

#include <avr/wdt.h>

#include <Thread.h>
#include <StaticThreadController.h>
#include <ThreadController.h>

#include <LiquidCrystal.h>

// Definition of sensors pins
LiquidCrystal lcd(12, 11, 5, 4, 6, 2);

#define ledPIN_BLUE 10
#define ledPIN_RED A2

#define JOYSTICK_X A0
#define JOYSTICK_Y A1
#define JOYSTICK_BUTTON 7

#define pinTrigger 8
#define pinEcho 9

#define buttonPin 3

#define DHTPIN 13

```

Then you have to assign each one in the setup() if they are an input or output pin:
``` cpp
void setup() {
  Serial.begin(9600);  //iniciar puerto serie

  pinMode(ledPIN_RED, OUTPUT);  
  pinMode(ledPIN_BLUE, OUTPUT);  

  pinMode(DHTPIN, INPUT); 

  pinMode(pinTrigger, OUTPUT);  
  pinMode(pinEcho, INPUT);      

  pinMode(buttonPin, INPUT_PULLUP);

  // we configure the push button pin as an input with pullup
  pinMode(JOYSTICK_BUTTON, INPUT_PULLUP);

  dht.begin();

  lcd.begin(16, 2);

  startTime = millis();

  startSequence();

  buttonThread.enabled = true;
  buttonThread.setInterval(1000);
  buttonThread.onRun(resetblink);

  threadcontroller.add(&buttonThread);

  // We deactivate the watchdog
  wdt_disable();
  wdt_enable(WDTO_8S);
}
```

The next step I decided to do was to make each menu in a general way, in which there was only Serial.print to see a basic operation. I also decided to include turning the LEDs on and off, since it was something simple and would encourage me to continue. Once I had the structure, I developed each of the parts of the menu separately, executing only each part of the menu, such as the part that allows us to dynamically see the temperature and humidity, the part of changing the prices, the part of the accountant, etc. it only executed one by calling it in the main loop constantly.

Once they all worked separately, I decided to put them together in a function that had a switch and depending on the one I received, which depended on the option chosen by the Joystick previously, I executed one function or another

``` cpp

// performAdminAction(): Executes specific actions in the admin mode
void performAdminAction(int option) {
  switch (option) {
    case 0:
      // Display dynamic temperature and humidity */
      showDinamicTemperatureAndHumidity();
      break;

    case 1:
      // Display distance measured by the ultrasonic sensor */
      showDistance();
      break;

    case 2:
      // Display the elapsed time since it does that since the board is booted */
      showCounter();
      break;

    case 3:
      // Allow the administrator to modify product prices */
      changePrices();
      break;
  }
}
```
Once I had all the menus and their functions created and functional, I only had to create a way to detect if the button has been pressed to change from admin to service or vice versa, or if you want to restart the service state. To do this, I decided to create a Thread that would call a function in intervals of 100 milliseconds, which checks whether the button has been pressed or not, calculates the time it has been pressed and acts from there.

``` cpp

#include <Thread.h>
#include <StaticThreadController.h>
#include <ThreadController.h>

//These are the thread that is responsible for controlling whether the button has been pressed and its controller.
Thread buttonThread = Thread();
ThreadController threadcontroller = ThreadController();

void setup(){
  buttonThread.enabled = true;
  buttonThread.setInterval(1000);
  buttonThread.onRun(resetblink);

  threadcontroller.add(&buttonThread);
}

void lopp(){
 /*
  We execute the thread controller. In this case it only launches one, which is the one that checks
  if the button has been pressed or not.
  */
  threadcontroller.run();
}
```

#### Problems
One problem I had was that the temperature and humidity sensor caused a short circuit, causing the LCD, ultrasound, and an LED not to work properly. After a few hours I managed to solve it after trying another type of code and checking all the cables and components.
Here I attach a photo of what the LCD showed me sometimes
![temperature_sensor_error](https://github.com/Ruben249/Vending-Machine/assets/102288264/f3b22a54-4fb7-4266-9ee0-e8642554f022)


Another problem was that the function that prints the LCD waiting for the Client was constantly executed, which caused errors when printing other things on the LCB. It was also difficult for me to find it, since it was constantly called from another function without me wanting it, but in the end I solved it.

First I decided to do everything with whiles, which at first seemed simpler, but created the possibility of getting stuck in one of them, since taking into account that the loop is a while, I ended up having 3 nested whiles, so I decided to change everything and add the boolean variables to only execute the part of the code that I wanted.

I decided to add a Watch Dog, to prevent it from getting stuck somewhere in the code. The mistake I made at the beginning was that in the lopp I put wdt_enable(WDTO_8S); so it always restarted, until I realized and put it in the setup and in the loop only wdt_reset() remained.

Another problem I had was that sometimes the ultrasound reader did not correctly detect whether there was a client or not, so I decided to try removing the cables from the way and that's how I got it to read it perfectly. Here I attach a photo of the circuit final with all its modifications:
![ultrasonic_image](https://github.com/Ruben249/Vending-Machine/assets/102288264/7174bd58-9284-468f-9999-08218e337d0a)
![final_circuit](https://github.com/Ruben249/Vending-Machine/assets/102288264/dbdec29a-7850-4e18-b31b-9dc6fbf8202f)


#### Final
My code consists of a single loop, the loop from which we check if we are admin or client, and also if it is executed if the button is pressed x time. Depending on whether we are an administrator or a client, we go into one menu or another and in each iteration, through some variables, we check which action to execute. Here I leave a fragment of the code with the boolean variables.

``` cpp
volatile bool admin_mode = false;

// With this boolean we check if the joystick button has been pressed
bool pressed_button = false;

// With this boolean we display only one time "Elija producto"
bool starter_controller = false;

// With this boolean we prevent prices always being saved in the auxiliary array
bool prices_controller = false;

// With this boolean we keep in the product selection menu.
bool drink_controller = false;
```

As you can see below, in the loop we only restart the watchdog, call the thread controller that executes the thread that checks if the button has been pressed and check if we are admin or service:

``` cpp
// Main loop to switch between service and admin modes
void loop() {
  /*
  If after 8 seconds it has not gone through the main loop again, 
  it means that it has gotten stuck somewhere, so we use the watchdog to avoid it.
  */
  wdt_reset();

  /*
  We execute the thread controller.
  In this case it only launches one, which is the one that checks if the button has been pressed or not.
  */
  threadcontroller.run();
  if (admin_mode == false) {
    exitAdminMode();
    serviceMode();
  } 
  else {
    enterAdminMode();
    adminMode();
  }
}
```
