# Setup
1. Install the [Arduino IDE](https://www.arduino.cc/en/software/).
2. Open the Arduino IDE.
3. Click on the `Boards Manager` icon in the left navigation menu.
4. Search for `esp32` and install `esp32 by Esspressif Systems >3.3.3` (if it fails, try again).
5. Plug your board into a USB port on your compupter. 
6. Select `ESP32 Family Device` from the `Select Board` drop down at the top of the IDE OR Plug your device out and in to find the port if the name doesn't show and select `ESP32 Dev Module`.
7. Click on `Tools/Port` and make sure a COM port is selected with the ESP32 Family Device.
8. Click on `Tools/Board` and select `ESP32 Wrover Module`
9. Make sure `Tools/Events Run On` and `Tools/Arduino Runs on` are set to run on `Core 0`
10. Avoid changing any other board settings at this time - if you have issues ask the host to double check your board config.
11. Click on the `Upload` arrow at the top of the IDE to flash the empty script onto the baord.
12. Power cycle the board - the display should be blank.
13. Replace the code with the following code and flash again:
    ```c++
    void setup() {
      Serial.begin(9600);
    }
    
    void loop() {
      Serial.println("hello world");
      delay(500);
    }
    ```
14. Click on the `Serial Monitor` 🔎 icon at the top right of the IDE and make sure you see `hello world` printed out.


# Testing the camera
1. Open example `File/Examples/ESP32/Camera/CameraWebServer`.
2. In `CameraWebServer.ino` comment out `#include "board_config.h"`.
3. In `app_httpd.cpp` comment out `#include "board_config.h"`.
4. Paste the following after the #includes at the top of `CameraWebServer.ino`:
    If your device has a mic.
    ```c++
    // Pins as per the TTGO-Camera pinout diagram.
    #define PWDN_GPIO_NUM  -1
    #define RESET_GPIO_NUM -1
    #define XCLK_GPIO_NUM  4
    #define SIOD_GPIO_NUM  18
    #define SIOC_GPIO_NUM  23
    
    #define Y9_GPIO_NUM    36
    #define Y8_GPIO_NUM    15
    #define Y7_GPIO_NUM    12
    #define Y6_GPIO_NUM    39
    #define Y5_GPIO_NUM    35
    #define Y4_GPIO_NUM    14
    #define Y3_GPIO_NUM    13
    #define Y2_GPIO_NUM    34
    #define VSYNC_GPIO_NUM 5
    #define HREF_GPIO_NUM  27
    #define PCLK_GPIO_NUM  25
    
    #define LED_GPIO_NUM -1
    ```
    If your device has no mic.
    ```
    #define PWDN_GPIO_NUM  26
    #define RESET_GPIO_NUM -1
    #define XCLK_GPIO_NUM 32
    #define SIOD_GPIO_NUM  13
    #define SIOC_GPIO_NUM  12
    
    #define Y9_GPIO_NUM    39
    #define Y8_GPIO_NUM    36
    #define Y7_GPIO_NUM    23
    #define Y6_GPIO_NUM    18
    #define Y5_GPIO_NUM    15
    #define Y4_GPIO_NUM    4
    #define Y3_GPIO_NUM    14
    #define Y2_GPIO_NUM    5
    #define VSYNC_GPIO_NUM 27
    #define HREF_GPIO_NUM  25
    #define PCLK_GPIO_NUM  19
    
    #define LED_GPIO_NUM -1
    ```
5. Fill in your Wifi `ssid` and `password`.
6. Click on the `Upload` arrow at the top of the IDE to flash the script onto the baord.
7. Open the serial monitor:
    1. Change the `Baud Rate` (top right of the serial monitor panel) to `115200`.
    2. Click the `Clear Output` button (just above the `Baud Rate` drop down.
    3. Press the `RST` button on your board (bottom left).
    4. You should see:
        ```
        WiFi connecting....
        WiFi connected
        Camera Ready! Use 'http://xxx.xxx.xxx.xxx' to connect
        ```
8. Copy the URL from the serial monitor into your browser.
9. Click `Get Still` to confirm that the camera is working.
10. You can play with the settings. Some settings may cause your app to crash - restart and try again. 

#Arduino Cloud Setup
1. Sign up for [Arduino Cloud](https://cloud.arduino.cc/).
2. Skip out of the automated setup.
3. Click on `Devices` in the left navigation menu.
4. Click on `ADD DEVICE`.
5. Click on `Compatible device`.
6. Select `ESP32` and `ESP32 Wrover Module`.
7. Name your device `smart_meter_device` and click the check mark ✔️.
8. Download and save your `Device ID` and `Secret Key`.
9. Click on `Things` in the left navigation menu.
10. Click `Create Thing` ➕.
11. Rename thing to `smart_meter_thing`.
12. Click `Add` Coud variable:
    1. Name: `flashMeterAmp`.
    2. Type: `Electrical Current`.
    3. Variable Permission: `Read Only`.
    4. Variable Update Policy: `On Change`.

#Arduino Cloud connection
1. In the Arduino IDE, click on the `Library Manager` in the left navigation panel and search for and install `ArduinoIoTCloud` with all dependancies.
2. Make a new sketch (`File/New Sketch`) and save it on your desktop with the name `smart_meter`. This will create a new folder on your desktop called `smart_meter` and inside that will be a file `smart_meter.ino`.
3. Make sure you have file extensions visible in explorer/finder.
4. Create a copy of `smart_meter.ino` and rename it to `thingProperties.h`. Make SURE the extension has changed. This file will not be visible in the Arduino IDE.
5. Replace the contents of each file with the files below:

   `smart_meter.ino`
   
    ```c++
    #include "thingProperties.h"
    
    void setup() {
    
      Serial.begin(9600);
      delay(1500); 
    
      initProperties();
    
      ArduinoCloud.begin(ArduinoIoTPreferredConnection);
      
      setDebugMessageLevel(2);
      ArduinoCloud.printDebugInfo();
    }
    
    int i = 0;
    
    void loop() {
      ArduinoCloud.update();
      delay(1000);
      flashMeterAmp = (i==0 ? i++: i--);
    }
    ```
 
    `thingProperties.h`
   
    ```c++
    #include <ArduinoIoTCloud.h>
    #include <Arduino_ConnectionHandler.h>
    
    const char DEVICE_LOGIN_NAME[]  = "*********";    // Copy from downloaded PDF
    const char DEVICE_KEY[]  = "****************";    // Copy from downloaded PDF
    
    const char SSID[]               = "*********";    // Network SSID (name)
    const char PASS[]               = "*********";    // Network password (use for WPA, or use as key for WEP)
   
    CloudElectricCurrent flashMeterAmp;
    
    void initProperties(){
    
      ArduinoCloud.setBoardId(DEVICE_LOGIN_NAME);
      ArduinoCloud.setSecretDeviceKey(DEVICE_KEY);
      ArduinoCloud.addProperty(flashMeterAmp, READ, ON_CHANGE, NULL);
    
    }
    
    WiFiConnectionHandler ArduinoIoTPreferredConnection(SSID, PASS);
   ```
7. 
