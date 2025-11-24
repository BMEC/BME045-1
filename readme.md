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
10. Set `Core Debug Level` to `Info`.
11. Avoid changing any other board settings at this time - if you have issues ask the host to double check your board config.
12. Click on the `Upload` arrow at the top of the IDE to flash the empty script onto the baord.
13. Power cycle the board - the display should be blank.
14. Replace the code with the following code and flash again:
    ```c++
    void setup() {
      Serial.begin(9600);
    }
    
    void loop() {
      Serial.println("hello world");
      delay(500);
    }
    ```
15. Click on the `Serial Monitor` 🔎 icon at the top right of the IDE and make sure you see `hello world` printed out.


# Testing the camera
1. Open example `File/Examples/ESP32/Camera/CameraWebServer`.
2. In `CameraWebServer.ino` comment out `#include "board_config.h"`.
3. In `app_httpd.cpp` comment out `#include "board_config.h"`.
4. Paste the following after the #includes at the top of `CameraWebServer.ino`:
    If your device has a mic.
    ```c++
    // Pins as per the board silk - note the diagram is incorrect.
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
    ```c++
    // Pins as per the diagram.
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
    1. Name: `powerW`.
    2. Type: `Power`.
    3. Variable Permission: `Read Only`.
    4. Variable Update Policy: `On Change`.
13. Click `Add` Coud variable:
    1. Name: `consumptionWh`.
    2. Type: `Floating Point Number`.
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
   
    float consumptionWh;
    CloudPower powerW;
    
    void initProperties(){
    
      ArduinoCloud.setBoardId(DEVICE_LOGIN_NAME);
      ArduinoCloud.setSecretDeviceKey(DEVICE_KEY);
      ArduinoCloud.addProperty(consumptionWh, READ, ON_CHANGE, NULL);
      ArduinoCloud.addProperty(powerW, READ, ON_CHANGE, NULL);
    
    }
    
    WiFiConnectionHandler ArduinoIoTPreferredConnection(SSID, PASS);
   ```

# Dashboard creations
1. Click on `Dashboards` in the left navigation menu.
2. Click on `Create dashboard` ➕.
3. Rename the dashboard to `Smart Meter Dashboard`.
4. Click `Edit` ✏️ at the top right.
5. Click `Add`.
6. Select `Chart`.
7. Click `Link Variable`, select `flashMeterAmp` and click `LINK VARIABLE` and then `DONE`.
8. You should see the current bounce between 0 and 1 🚀.

# Meter firmware
1. Replace the code in `smart_meter.ino` with the code below.
2. Comment out the appropriate camera pins.
3. Have fun adding to your dashboard and testing your code!
4. Work on extra features like:
    1. Showing the data on the display.
    2. Implementing flash memory so your data is saved if you loose power.
    3. Auto reset the consumption on the 1st of the month - or add monthly consumption.
```c++
// Copyright (c) 2025 BMEC Technologies. All rights reserved.

// Source code for IoT smart meter for prepaid meters.
// See https://github.com/BMEC/BME045-1 for details.

#include "thingProperties.h"
#include "esp_camera.h"

// If your device has a mic.
// Pins as per the board silk - note the diagram is incorrect.
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

// If your device has no mic.
// Pins as per the diagram.
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

// FEATURE implement flash to store usage.
// FEATURE reset usage on first of month.

// Change in average pixel brighness required to trigger a flash.
static const int32_t flashThreshold = 50;

// Watts per hour per meter flash.
static const float wattHourPerFlash = 1.0;

// Average pixel brightness of the previous frame.
int32_t previousBrightnessAverage = 0;

// Time since last flash in micro-seconds since boot.
int64_t previousFlashTimeUs = 0;

/// This task initialises the camera and then runs the camera and smart meter loop.
void meterTask() {

  // Camera config.
  camera_config_t cameraConfig;
  cameraConfig.pin_d0 = Y2_GPIO_NUM;
  cameraConfig.pin_d1 = Y3_GPIO_NUM;
  cameraConfig.pin_d2 = Y4_GPIO_NUM;
  cameraConfig.pin_d3 = Y5_GPIO_NUM;
  cameraConfig.pin_d4 = Y6_GPIO_NUM;
  cameraConfig.pin_d5 = Y7_GPIO_NUM;
  cameraConfig.pin_d6 = Y8_GPIO_NUM;
  cameraConfig.pin_d7 = Y9_GPIO_NUM;
  cameraConfig.pin_xclk = XCLK_GPIO_NUM;
  cameraConfig.pin_pclk = PCLK_GPIO_NUM;
  cameraConfig.pin_vsync = VSYNC_GPIO_NUM;
  cameraConfig.pin_href = HREF_GPIO_NUM;
  cameraConfig.pin_sccb_sda = SIOD_GPIO_NUM;
  cameraConfig.pin_sccb_scl = SIOC_GPIO_NUM;
  cameraConfig.pin_pwdn = PWDN_GPIO_NUM;
  cameraConfig.pin_reset = RESET_GPIO_NUM;
  cameraConfig.xclk_freq_hz = 20000000;
  cameraConfig.pixel_format = PIXFORMAT_GRAYSCALE;
  cameraConfig.fb_location = CAMERA_FB_IN_PSRAM;
  cameraConfig.frame_size = FRAMESIZE_QVGA;
  cameraConfig.grab_mode = CAMERA_GRAB_LATEST;
  cameraConfig.fb_count = 1;

  // Camera init.
  esp_err_t err = esp_camera_init(&cameraConfig);
  if (err != ESP_OK) {
    log_e("Camera init failed with error: 0x%x - %s",
          err,
          esp_err_to_name(err));
    log_e("Retry in 2 seconds...");
    delay(2000);
    ESP.restart();
  }

  // Loop indefinitely.
  while (true) {
    // Delay 50 ms to give other tasks a chance.
    delay(50);

    // Instantiate the frame buffer.
    camera_fb_t* frameBuffer;

    // Read image from camera into buffer.
    frameBuffer = esp_camera_fb_get();

    // Check for valid frame and skip to next loop if failed.
    if (!frameBuffer) {
      log_w("webSocketTask: Camera capture failed");
      return;
    }

    int32_t brightnessAverage = 0;
    uint32_t pixelIndex = -1;

    // Sum the value of all pixels.
    while (++pixelIndex < frameBuffer->len) {
      brightnessAverage += ((uint8_t*)frameBuffer->buf)[pixelIndex];
    }

    // Compute the average.
    brightnessAverage /= pixelIndex;

    // Release the frame buffer.
    esp_camera_fb_return(frameBuffer);

    // Debugging log to see the brightness in every loop.
    log_v("brightnessAverage %ld", brightnessAverage);

    // Detetect flash.
    if ((brightnessAverage - previousBrightnessAverage) > flashThreshold) {

      // Add the usage to the consumption.
      consumptionWh += wattHourPerFlash;

      // Get the flash time.
      int64_t flashTimeUs = esp_timer_get_time();

      // Power in watts is watthoursPerflash devided by the time since the last flash.
      // The time delta needs to be converted from micro-seconds to hours.
      powerW = wattHourPerFlash / (((float)(flashTimeUs - previousFlashTimeUs)) / 1000.0 / 1000.0 / 60.0 / 60.0);

      // Update the previous flash time.
      previousFlashTimeUs = flashTimeUs;

      log_i("**Flash detected**\nconsumptionWh, %.2f \tpowerW, %.2f", consumptionWh, (float)powerW);
    }

    // Update the previous brightness.
    previousBrightnessAverage = brightnessAverage;
  }
}

void setup() {

  // Enable Serial.
  Serial.begin(9600);

  // Arduino Cloud init.
  initProperties();

  // Start Arduino Cloud.
  ArduinoCloud.begin(ArduinoIoTPreferredConnection);

  // Set Arduino cloud log level.
  setDebugMessageLevel(2);

  // Print Arduino Cloud debug info.
  ArduinoCloud.printDebugInfo();

  // Create and start the meterTask.
  (void)xTaskCreatePinnedToCore([](void* nullRef) {
    meterTask();
  },
                                "meterTask", 5000, nullptr, 6, nullptr, 1);
}


void loop() {
  // Service Arduino Cloud.
  ArduinoCloud.update();
  // Delay 100 ms to give other tasks a chance.
  delay(100);
}
```

