# Setup
1. Install the [Arduino IDE](https://www.arduino.cc/en/software/).
2. Open the Arduino IDE.
3. Click on the `Boards Manager` icon in the left navigation menu.
4. Search for `esp32` and install `esp32 by Esspressif Systems >3.3.3` (if it fails, try again).
5. Plug your board into a USB port on your compupter. 
6. Select `ESP32 Family Device` from the `Select Board` drop down at the top of the IDE OR Plug your device out and in to find the port if the name doesn't show and select `ESP32 Dev Module`.
7. Click on `Tools/Port` and make sure a COM port is selected with the ESP32 Family Device.
8. Click on `Tools/Board` and select `ESP32 Wrover Module` - avoid changing any other board settings at this time.
9. Click on the `Upload` arrow at the top of the IDE to flash the empty script onto the baord.
10. Power cycle the board - the display should be blank.
11. Replace the code with the following code and flash again:
    ```c++
    void setup() {
      Serial.begin(9600);
    }
    
    void loop() {
      Serial.println("hello world");
      delay(500);
    }
    ```
12. Click on the `Serial Monitor` 🔎 icon at the top right of the IDE and make sure you see `hello world` printed out.

