# MoveItYourself
### il movimento per tutti!

Siamo partiti da un'idea: far muovere una macchinina telecomandata attraverso l'utilizzo di un giroscopio. 

Successivamente, ci siamo resi conto che il progetto poteva essere sviluppato per bambini e bambine con disabilità motorie, quindi, non in grado di giocare con determinati giocattoli per problematiche di accessibilità e utilizzo del controller.

Abbiamo quindi sviluppato l'idea di una startup che creerà degli strumenti e dei supporti per far si che anche una persona con disabilità fisiche e cognitive riesca a far muovere oggetti come una macchinina telecomandata.


test vari con arduino e tinkerkit linear potentiometer

bluethoot = manda tramite byte e non int (spiega bene sta cosa)

Codice arduino nano 33 iot centrale
  
  #include <ArduinoBLE.h> </p>
#include <Arduino_LSM6DS3.h> </p>
float x, y, z; </p>
int plusThreshold = 100; </p>
int minusThreshold = -100; </p>
void setup() { </p>
  Serial.begin(9600); </p>
    Serial.println("Started"); </p>
if (!IMU.begin()) { </p>
      Serial.println("Failed to initialize IMU!"); </p>
    while (1); </p>
  } </p>
  BLE.begin(); </p>
  BLE.scanForUuid("19b10000-e8f2-537e-4f6c-d104768a1214"); </p>
} </p>
int posX = 0; </p>
int posOldX = 0; </p>
int posY = 0; </p>
int posOldY = 0; </p>
void loop() { </p>
  BLEDevice peripheral = BLE.available(); </p>
  Serial.println("sto cercando"); </p>
  if (peripheral) { </p>
    // discovered a peripheral, print out address, local name, and advertised service </p>
    Serial.print("Found "); </p>
    Serial.print(peripheral.address());</p>
    Serial.print(" '"); </p>
    Serial.print(peripheral.localName());</p>
    Serial.print("' ");</p>
    Serial.print(peripheral.advertisedServiceUuid());</p>
    Serial.println(); </p>
    if (peripheral.localName() != "LED") {</p>
      return;</p>
    }</p>
    // stop scanning</p>
    BLE.stopScan();</p>
    controlLed(peripheral);</p>
    // peripheral disconnected, start scanning again </p>
    BLE.scanForUuid("19b10000-e8f2-537e-4f6c-d104768a1214");</p>
  }</p>
}</p>
void controlLed(BLEDevice peripheral) {</p>
  // connect to the peripheral</p>
  Serial.println("Connecting ...");</p>
  if (peripheral.connect()) {</p>
    Serial.println("Connected");</p>
  } else {</p>
    Serial.println("Failed to connect!");</p>
    return;</p>
  }</p>
  // discover peripheral attributes</p>
  Serial.println("Discovering attributes ...");</p>
  if (peripheral.discoverAttributes()) {</p>
    Serial.println("Attributes discovered");</p>
  } else {</p>
    Serial.println("Attribute discovery failed!");</p>
    peripheral.disconnect();</p>
    return;</p>
  }</p>
  // retrieve the LED characteristic</p>
  BLECharacteristic ledCharacteristic = peripheral.characteristic("19b10001-e8f2-537e-4f6c-d104768a1214");</p>
  if (!ledCharacteristic) {</p>
    Serial.println("Peripheral does not have LED characteristic!");</p>
    peripheral.disconnect();</p>
    return;</p>
  } else if (!ledCharacteristic.canWrite()) {</p>
    Serial.println("Peripheral does not have a writable LED characteristic!");</p>
    peripheral.disconnect();</p>
    return;</p>
  }</p>
while (peripheral.connected()) {</p>
if (IMU.gyroscopeAvailable()) {</p>
IMU.readGyroscope(x, y, z);</p>
if (y > plusThreshold)</p>
{        posX--;</p>
      }</p>
      if (y < minusThreshold)</p>
      {</p>
        posX++;</p>
      }</p>
      if (x < minusThreshold)</p>
      {</p>
        posY++;</p>
      }</p>
      if (x > plusThreshold)</p>
      {</p>
        posY--;</p>
      }</p>
    }</p>
    if (posX > 1) {</p>
      posX = 1;</p>
    }</p>
    if (posX < -1) {</p>
      posX = -1;</p>
    }</p>
    if (posY > 1) {</p>
      posY = 1;</p>
    }</p>
    if (posY < -1) {</p>
      posY = -1;</p>
    }</p>
    if (posX != posOldX) {</p>
      ledCharacteristic.writeValue("X");</p>
      ledCharacteristic.writeValue((byte)posX);</p>
      Serial.println(posX);</p>
      delay(500);</p>
    }</p>
    if (posY != posOldY) {</p>
      ledCharacteristic.writeValue("Y");</p>
      ledCharacteristic.writeValue((byte)posY);</p>
      Serial.println(posY);</p>
      delay(500);</p>
    }</p>
    posOldX = posX;</p>
    posOldY = posY;</p>
  }
  Serial.println("Peripheral disconnected");</p>
}</p>
codice arduino nano 33 iot periferico</p>
#include <ArduinoBLE.h></p>
BLEService ledService("19B10000-E8F2-537E-4F6C-D104768A1214"); // Bluetooth® Low Energy LED Service</p>
// Bluetooth® Low Energy LED Switch Characteristic - custom 128-bit UUID, read and writable by central</p>
BLEByteCharacteristic switchCharacteristic("19B10001-E8F2-537E-4F6C-D104768A1214", BLERead | BLEWrite);</p>
void setup() {</p>
  Serial.begin(9600);</p>
  // set button pin to output mode</p>
  pinMode(6, OUTPUT);</p>
  pinMode(8, OUTPUT);</p>
  pinMode(10, OUTPUT);</p>
  pinMode(12, OUTPUT);</p>
  // begin initialization</p>
  if (!BLE.begin()) {</p>
    Serial.println("starting Bluetooth® Low Energy module failed!");</p>
    while (1);</p>
  }</p>
  // set advertised local name and service UUID:</p>
  BLE.setLocalName("LED");</p>
  BLE.setAdvertisedService(ledService);</p>
  // add the characteristic to the service</p>
  ledService.addCharacteristic(switchCharacteristic);</p>
  // add service</p>
  BLE.addService(ledService);</p>
  // set the initial value for the characeristic:</p>
  switchCharacteristic.writeValue(0);</p>
  // start advertising</p>
  BLE.advertise();</p>
  Serial.println("BLE LED Peripheral");</p>
}</p>
int inByte = 0;</p>
int switchByte = 0;</p>
void loop() {</p>
  // listen for Bluetooth® Low Energy peripherals to connect:</p>
  BLEDevice central = BLE.central();</p>
  // if a central is connected to peripheral:</p>
  if (central) {</p>
    Serial.print("Connected to central: ");</p>
    // print the central's MAC address:</p>
    Serial.println(central.address());</p>
    // while the central is still connected to peripheral:</p>
    while (central.connected()) {</p>
      // if the remote device wrote to the characteristic,</p>
      // use the value to control the LED:</p>
      if (switchCharacteristic.written()) {</p>
        inByte = switchCharacteristic.value();</p>
        Serial.println(inByte);</p>
        if (inByte == 88) {</p>
          switchByte = 0;</p>
        }</p>
        else if (inByte == 89) {</p>
          switchByte = 1;</p>
        }</p>
        else if (inByte == 1) {</p>
          if (switchByte == 1) {</p>
            digitalWrite(10, HIGH);</p>
          }</p>
          else {</p>
            digitalWrite(6, HIGH);</p>
          }</p>
        }</p>
        else if (inByte == 0) {</p>
          if (switchByte == 1) {</p>
            digitalWrite(10, LOW);</p>
            digitalWrite(8, LOW);</p>
          }</p>
          else {</p>
            digitalWrite(6, LOW);</p>
            digitalWrite(12, LOW);</p>
          }</p>
        }</p>
        else if (inByte == 255) {</p>
          if (switchByte == 1) {</p>
            digitalWrite(8, HIGH);</p>
          }</p>
          else {</p>
            digitalWrite(12, HIGH);</p>
          }</p>
        }</p>
      }</p>
    }
    // when the central disconnects, print it out:
    Serial.print(F("Disconnected from central: "));
    Serial.println(central.address());
  }
}
