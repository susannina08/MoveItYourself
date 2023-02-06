# MoveItYourself
### il movimento per tutti!

Siamo partiti da un'idea: far muovere una macchinina telecomandata attraverso l'utilizzo di un giroscopio. 

Successivamente, ci siamo resi conto che il progetto poteva essere sviluppato per bambini e bambine con disabilità motorie, quindi, non in grado di giocare con determinati giocattoli per problematiche di accessibilità e utilizzo del controller.

Abbiamo quindi sviluppato l'idea di una startup che creerà degli strumenti e dei supporti per far si che anche una persona con disabilità fisiche e cognitive riesca a far muovere oggetti come una macchinina telecomandata.


test vari con arduino e tinkerkit linear potentiometer

bluethoot = manda tramite byte e non int (spiega bene sta cosa)

Codice arduino nano 33 iot centrale
  
  #include <ArduinoBLE.h>
#include <Arduino_LSM6DS3.h>
float x, y, z;
int plusThreshold = 100;
int minusThreshold = -100;
void setup() {
  Serial.begin(9600);
    Serial.println("Started");
if (!IMU.begin()) {
      Serial.println("Failed to initialize IMU!");  
    while (1);
  }
  BLE.begin();
  BLE.scanForUuid("19b10000-e8f2-537e-4f6c-d104768a1214");
}
int posX = 0;
int posOldX = 0;
int posY = 0;
int posOldY = 0;
void loop() {
  BLEDevice peripheral = BLE.available();
  Serial.println("sto cercando");
  if (peripheral) {
    // discovered a peripheral, print out address, local name, and advertised service
    Serial.print("Found ");
    Serial.print(peripheral.address());
    Serial.print(" '");
    Serial.print(peripheral.localName());
    Serial.print("' ");
    Serial.print(peripheral.advertisedServiceUuid());
    Serial.println();
    if (peripheral.localName() != "LED") {
      return;
    }
    // stop scanning
    BLE.stopScan();
    controlLed(peripheral);
    // peripheral disconnected, start scanning again
    BLE.scanForUuid("19b10000-e8f2-537e-4f6c-d104768a1214");
  }
}
void controlLed(BLEDevice peripheral) {
  // connect to the peripheral
  Serial.println("Connecting ...");
  if (peripheral.connect()) {
    Serial.println("Connected");
  } else {
    Serial.println("Failed to connect!");
    return;
  }
  // discover peripheral attributes
  Serial.println("Discovering attributes ...");
  if (peripheral.discoverAttributes()) {
    Serial.println("Attributes discovered");
  } else {
    Serial.println("Attribute discovery failed!");
    peripheral.disconnect();
    return;
  }
  // retrieve the LED characteristic
  BLECharacteristic ledCharacteristic = peripheral.characteristic("19b10001-e8f2-537e-4f6c-d104768a1214");
  if (!ledCharacteristic) {
    Serial.println("Peripheral does not have LED characteristic!");
    peripheral.disconnect();
    return;
  } else if (!ledCharacteristic.canWrite()) {
    Serial.println("Peripheral does not have a writable LED characteristic!");
    peripheral.disconnect();
    return;
  }
while (peripheral.connected()) {
if (IMU.gyroscopeAvailable()) {
IMU.readGyroscope(x, y, z);
if (y > plusThreshold)
{        posX--;
      }
      if (y < minusThreshold)
      {
        posX++;
      }
      if (x < minusThreshold)
      {
        posY++;
      }
      if (x > plusThreshold)
      {
        posY--;
      }
    }
    if (posX > 1) {
      posX = 1;
    }
    if (posX < -1) {
      posX = -1;
    }
    if (posY > 1) {
      posY = 1;
    }
    if (posY < -1) {
      posY = -1;
    }
    if (posX != posOldX) {
      ledCharacteristic.writeValue("X");
      ledCharacteristic.writeValue((byte)posX);
      Serial.println(posX);
      delay(500);
    }
    if (posY != posOldY) {
      ledCharacteristic.writeValue("Y");
      ledCharacteristic.writeValue((byte)posY);
      Serial.println(posY);
      delay(500);
    }
    posOldX = posX;
    posOldY = posY;
  }
  Serial.println("Peripheral disconnected");
}

codice arduino nano 33 iot periferico
#include <ArduinoBLE.h>
BLEService ledService("19B10000-E8F2-537E-4F6C-D104768A1214"); // Bluetooth® Low Energy LED Service
// Bluetooth® Low Energy LED Switch Characteristic - custom 128-bit UUID, read and writable by central
BLEByteCharacteristic switchCharacteristic("19B10001-E8F2-537E-4F6C-D104768A1214", BLERead | BLEWrite);
void setup() {
  Serial.begin(9600);
  // set button pin to output mode
  pinMode(6, OUTPUT);
  pinMode(8, OUTPUT);
  pinMode(10, OUTPUT);
  pinMode(12, OUTPUT);
  // begin initialization
  if (!BLE.begin()) {
    Serial.println("starting Bluetooth® Low Energy module failed!");
    while (1);
  }
  // set advertised local name and service UUID:
  BLE.setLocalName("LED");
  BLE.setAdvertisedService(ledService);
  // add the characteristic to the service
  ledService.addCharacteristic(switchCharacteristic);
  // add service
  BLE.addService(ledService);
  // set the initial value for the characeristic:
  switchCharacteristic.writeValue(0);
  // start advertising
  BLE.advertise();
  Serial.println("BLE LED Peripheral");
}
int inByte = 0;
int switchByte = 0;
void loop() {
  // listen for Bluetooth® Low Energy peripherals to connect:
  BLEDevice central = BLE.central();
  // if a central is connected to peripheral:
  if (central) {
    Serial.print("Connected to central: ");
    // print the central's MAC address:
    Serial.println(central.address());
    // while the central is still connected to peripheral:
    while (central.connected()) {
      // if the remote device wrote to the characteristic,
      // use the value to control the LED:
      if (switchCharacteristic.written()) {
        inByte = switchCharacteristic.value();
        Serial.println(inByte);
        if (inByte == 88) {
          switchByte = 0;
        }
        else if (inByte == 89) {
          switchByte = 1;
        }
        else if (inByte == 1) {
          if (switchByte == 1) {
            digitalWrite(10, HIGH);
          }
          else {
            digitalWrite(6, HIGH);
          }
        }
        else if (inByte == 0) {
          if (switchByte == 1) {
            digitalWrite(10, LOW);
            digitalWrite(8, LOW);
          }
          else {
            digitalWrite(6, LOW);
            digitalWrite(12, LOW);
          }
        }
        else if (inByte == 255) {
          if (switchByte == 1) {
            digitalWrite(8, HIGH);
          }
          else {
            digitalWrite(12, HIGH);
          }
        }
      }
    }
    // when the central disconnects, print it out:
    Serial.print(F("Disconnected from central: "));
    Serial.println(central.address());
  }
}
