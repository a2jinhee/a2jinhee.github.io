---
layout: default
title: (Nano 33 BLE Sense) BLE Communication
nav_exclude: true
grand_parent: Projects
parent: ICE3028 Term Project
show_toc: true 
---

## (Nano 33 BLE Sense) BLE Communication 
{: .no_toc }

_2023 11.04_  
<br>

*reference*  

- Nano 33 BLE Sense Documentation: [Connecting Nano 33 BLE Devices over Bluetooth](https://docs.arduino.cc/tutorials/nano-33-ble-sense/ble-device-to-device)

--- 

1. TOC
{:toc}

--- 

## 1. What is Bluetooth® Low Energy?  
- Classic Bluetooth®: Includes three working modes- BR, EDR, and HS(AMP)
  - (ex) Audio applications(wireless headphones)
- Bluetooth® Low Energy: Designed to reduce the power consumption by reducing the amount of time that the Bluetooth radio is on. 
  - (ex) Power constrained applications, such as wearables and IoT devices
- Classic Bluetooth® and Bluetooth® Low Energy have different physical layer modulation and demodulation methods, **hence, they are not compatible**

## 2. How Does Bluetooth® Low Energy Work?  
- Central Role (= Servers): Performs a scan and listen for broadcasting information. The central device attempts to connect with peripheral device as soon as it picks up the advertising information.
- Peripheral Role (= Clients): When Bluetooth® connection is established, it will advertise or broadcast information to near devices. 
- Once a connection is established, the central device will interact with the available information that the peripheral device has. This information exchange is called **services**.

## 3. Services and Characteristics
- Service: A group of capabilities. 
  - (ex) A smartwatch can measure your heart rate, track physical activity and track your sleep patterns. These three capabilities, would exist in a service called health service.
- UUID: A unique identification code given to every service. 16-bit or 32-bit long for official Bluetooth® services while non-official Bluetooth® services are 128-bit long.  
- Characteristics: Each characteristic represents a unique capability of the central device. The peripheral device can  
  - 1) write information to,   
  - 2) request information from,   
  - 3) and subscribe to updates from these characteristics.    
  Any characteristic, like the services, have a 16 bit long or 128 bit long UUID.

## 4. Using Bluetooth® Low Energy and Arduino
- Central: Nano 33 BLE Sense (to use the embedded gesture sensor) 
- Peripheral: Nano 33 BLE 
- Service: gestureService
- Characteristic: gesture_type 
- Summary: If the central device detects a gesture with its gesture sensor, it will write the type of the gesture detected in the gesture_type characteristic of the gestureService. Then, based on the value stored in the gesture_type characteristic, the built-in RGB LED of the peripheral device will turn on a specific color.  

### Programming the Central Device
```cpp
/*
  BLE_Central_Device.ino
  https://docs.arduino.cc/tutorials/nano-33-ble-sense/ble-device-to-device

  This program uses the ArduinoBLE library to set-up an Arduino Nano 33 BLE Sense
  as a central device and looks for a specified service and characteristic in a
  peripheral device. If the specified service and characteristic is found in a
  peripheral device, the last detected value of the on-board gesture sensor of
  the Nano 33 BLE Sense, the APDS9960, is written in the specified characteristic.

  The circuit:
  - Arduino Nano 33 BLE Sense.

  This example code is in the public domain.
*/

#include <ArduinoBLE.h>
#include <Arduino_APDS9960.h>

// Unique UUID code for service
const char *deviceServiceUuid = "19b10000-e8f2-537e-4f6c-d104768a1214";
// Unique UUID code for characteristic
const char *deviceServiceCharacteristicUuid = "19b10001-e8f2-537e-4f6c-d104768a1214";

int gesture = -1;
int oldGestureValue = -1;

void setup()
{
  Serial.begin(9600);
  while (!Serial)
    ;

  // Initialize the APDS9960 sensor for gesture recognition
  if (!APDS.begin())
  {
    Serial.println("* Error initializing APDS9960 sensor!");
  }

  APDS.setGestureSensitivity(80);

  // Initialize BLE
  if (!BLE.begin())
  {
    Serial.println("* Starting Bluetooth® Low Energy module failed!");
    while (1)
      ;
  }

  /*
  // Set the name of the central device,
  // and start advertising to make itself
  // discoverable to other peripheral devices
  */
  BLE.setLocalName("Nano 33 BLE (Central)");
  BLE.advertise();

  Serial.println("Arduino Nano 33 BLE Sense (Central Device)");
  Serial.println(" ");
}

void loop()
{
  connectToPeripheral();
}

void connectToPeripheral()
{
  BLEDevice peripheral;

  Serial.println("- Discovering peripheral device...");

  // Scan for a peripheral with a specific service UUID
  // find available device and store in 'peripheral'
  do
  {
    BLE.scanForUuid(deviceServiceUuid);
    peripheral = BLE.available();
  } while (!peripheral);

  // Peripheral Info
  if (peripheral)
  {
    Serial.println("* Peripheral device found!");
    Serial.print("* Device MAC address: ");
    Serial.println(peripheral.address());
    Serial.print("* Device name: ");
    Serial.println(peripheral.localName());
    Serial.print("* Advertised service UUID: ");
    // This should be same as central service uuid
    Serial.println(peripheral.advertisedServiceUuid());
    Serial.println(" ");
    BLE.stopScan(); // stop scan when found

    // Control the peripheral once found
    controlPeripheral(peripheral);
  }
}

void controlPeripheral(BLEDevice peripheral)
{
  // Check if peripheral is connected
  Serial.println("- Connecting to peripheral device...");
  if (peripheral.connect())
  {
    Serial.println("* Connected to peripheral device!");
    Serial.println(" ");
  }
  else
  {
    Serial.println("* Connection to peripheral device failed!");
    Serial.println(" ");
    return;
  }

  // Check for peripheral's services and characteristics.
  Serial.println("- Discovering peripheral device attributes...");
  if (peripheral.discoverAttributes())
  {
    Serial.println("* Peripheral device attributes discovered!");
    Serial.println(" ");
  }
  else
  {
    Serial.println("* Peripheral device attributes discovery failed!");
    Serial.println(" ");
    peripheral.disconnect();
    return;
  }

  // Store peripheral's characteristic container to central's characteristic
  BLECharacteristic gestureCharacteristic = peripheral.characteristic(deviceServiceCharacteristicUuid);

  // Check if peripheral device initialized/writable characteristic
  if (!gestureCharacteristic)
  {
    Serial.println("* Peripheral device does not have gesture_type characteristic!");
    peripheral.disconnect();
    return;
  }
  else if (!gestureCharacteristic.canWrite())
  {
    Serial.println("* Peripheral does not have a writable gesture_type characteristic!");
    peripheral.disconnect();
    return;
  }

  // Write value to gesture_type characteristic
  while (peripheral.connected())
  {
    gesture = gestureDetectection();
    // Check if gesture changed from before
    if (oldGestureValue != gesture)
    {
      oldGestureValue = gesture;
      Serial.print("* Writing value to gesture_type characteristic: ");
      Serial.println(gesture);
      gestureCharacteristic.writeValue((byte)gesture);
      Serial.println("* Writing value to gesture_type characteristic done!");
      Serial.println(" ");
    }
  }
  Serial.println("- Peripheral device disconnected!");
}

// Gesture detection function
int gestureDetectection()
{
  if (APDS.gestureAvailable())
  {
    gesture = APDS.readGesture();

    switch (gesture)
    {
    case GESTURE_UP:
      Serial.println("- UP gesture detected");
      break;
    case GESTURE_DOWN:
      Serial.println("- DOWN gesture detected");
      break;
    case GESTURE_LEFT:
      Serial.println("- LEFT gesture detected");
      break;
    case GESTURE_RIGHT:
      Serial.println("- RIGHT gesture detected");
      break;
    default:
      Serial.println("- No gesture detected");
      break;
    }
  }
  return gesture;
}
```

### Programming the Peripheral Device 
```cpp
/*
  BLE_Peripheral.ino

  This program uses the ArduinoBLE library to set-up an Arduino Nano 33 BLE
  as a peripheral device and specifies a service and a characteristic. Depending
  of the value of the specified characteristic, an on-board LED gets on.

  The circuit:
  - Arduino Nano 33 BLE.

  This example code is in the public domain.
*/

#include <ArduinoBLE.h>

enum
{
  GESTURE_NONE = -1,
  GESTURE_UP = 0,
  GESTURE_DOWN = 1,
  GESTURE_LEFT = 2,
  GESTURE_RIGHT = 3
};

// Unique UUID code for service
const char *deviceServiceUuid = "19b10000-e8f2-537e-4f6c-d104768a1214";
// Unique UUID code for characteristic
const char *deviceServiceCharacteristicUuid = "19b10001-e8f2-537e-4f6c-d104768a1214";

int gesture = -1;

// Define a BLE service and characteristic !! (done in peripheral)
BLEService gestureService(deviceServiceUuid);
BLEByteCharacteristic gestureCharacteristic(deviceServiceCharacteristicUuid, BLERead | BLEWrite);

void setup()
{
  Serial.begin(9600);
  while (!Serial)
    ;
  // Initialize LEDs and set their initial states
  pinMode(LEDR, OUTPUT);
  pinMode(LEDG, OUTPUT);
  pinMode(LEDB, OUTPUT);
  pinMode(LED_BUILTIN, OUTPUT);
  digitalWrite(LEDR, HIGH);
  digitalWrite(LEDG, HIGH);
  digitalWrite(LEDB, HIGH);
  digitalWrite(LED_BUILTIN, LOW);

  // Initialize the BLE module
  if (!BLE.begin())
  {
    Serial.println("- Starting Bluetooth® Low Energy module failed!");
    while (1)
      ;
  }

  /*
  // Set the name of the peripheral device,
  // and define the services and characteristics
  */
  BLE.setLocalName("Arduino Nano 33 BLE (Peripheral)");
  // the service that the peripheral will advertise
  // used to specify which service you want to actively advertise to other devices.
  BLE.setAdvertisedService(gestureService);
  // adds a characteristic to 'gestureService'
  gestureService.addCharacteristic(gestureCharacteristic);
  // adds 'gestureService' to the list of services provided by the peripheral, necessary to expose the service to central devices
  // registers the service with the BLE stack, making it available for central devices to discover when they connect to the peripheral.
  BLE.addService(gestureService);
  // initial value of 'gestureCharacteristic' set to -1
  gestureCharacteristic.writeValue(-1);
  // This function call starts the advertising process
  BLE.advertise();

  Serial.println("Nano 33 BLE (Peripheral Device)");
  Serial.println(" ");
}

void loop()
{
  // Checks for a central device's connection
  BLEDevice central = BLE.central();
  Serial.println("- Discovering central device...");
  delay(500);

  if (central)
  {
    Serial.println("* Connected to central device!");
    Serial.print("* Device MAC address: ");
    Serial.println(central.address());
    Serial.println(" ");

    while (central.connected())
    {
      // Check if characteristic was written to by central
      if (gestureCharacteristic.written())
      {
        // Update the LEDs based on the received gesture value
        gesture = gestureCharacteristic.value();
        writeGesture(gesture);
      }
    }

    Serial.println("* Disconnected to central device!");
  }
}

// Handle changes in the characteristic's value and update LEDs
void writeGesture(int gesture)
{
  Serial.println("- Characteristic <gesture_type> has changed!");

  switch (gesture)
  {
  case GESTURE_UP:
    Serial.println("* Actual value: UP (red LED on)");
    Serial.println(" ");
    digitalWrite(LEDR, LOW);
    digitalWrite(LEDG, HIGH);
    digitalWrite(LEDB, HIGH);
    digitalWrite(LED_BUILTIN, LOW);
    break;
  case GESTURE_DOWN:
    Serial.println("* Actual value: DOWN (green LED on)");
    Serial.println(" ");
    digitalWrite(LEDR, HIGH);
    digitalWrite(LEDG, LOW);
    digitalWrite(LEDB, HIGH);
    digitalWrite(LED_BUILTIN, LOW);
    break;
  case GESTURE_LEFT:
    Serial.println("* Actual value: LEFT (blue LED on)");
    Serial.println(" ");
    digitalWrite(LEDR, HIGH);
    digitalWrite(LEDG, HIGH);
    digitalWrite(LEDB, LOW);
    digitalWrite(LED_BUILTIN, LOW);
    break;
  case GESTURE_RIGHT:
    Serial.println("* Actual value: RIGHT (built-in LED on)");
    Serial.println(" ");
    digitalWrite(LEDR, HIGH);
    digitalWrite(LEDG, HIGH);
    digitalWrite(LEDB, HIGH);
    digitalWrite(LED_BUILTIN, HIGH);
    break;
  default:
    digitalWrite(LEDR, HIGH);
    digitalWrite(LEDG, HIGH);
    digitalWrite(LEDB, HIGH);
    digitalWrite(LED_BUILTIN, LOW);
    break;
  }
}
```