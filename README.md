# **SMokeNode V1 DIY ESP8266 Project | Smart Smoke & Gas Alarm with OLED Display**

## **Technical Documentation and User Manual (Version 2.0)**

SmokeNode is an IoT-based smart home security node that monitors flammable gas, smoke, and LPG levels in real time. Powered by an integrated Wi-Fi module, it provides atomic real-time synchronization over the internet (NTP). In case of an emergency, it generates alarm logs and permanently stores them in its internal flash memory (LittleFS), ensuring no data is lost even if the device is powered down.

📺 **Watch the Full Project Video on YouTube:** <https://youtu.be/1z90qu-z850>

## **1\. Hardware Components and Pin Configurations**

To ensure stable and uninterrupted operation, the ESP8266 (NodeMCU) based hardware architecture is configured as follows:

### **SmokeNode Component Pinout & Connection Diagram**

| **Component Name**       | **Component Pin**        | **ESP8266 (NodeMCU) Equivalent** | **Connection Type / Purpose**              |
| ------------------------ | ------------------------ | -------------------------------- | ------------------------------------------ |
| **MQ-2 Gas Sensor**      | AO (Analog Out)          | A0                               | Analog Gas Measurement Signal              |
| ---                      | ---                      | ---                              | ---                                        |
|                          | GND                      | GND                              | Ground (-)                                 |
| ---                      | ---                      | ---                              | ---                                        |
|                          | VCC                      | VIN (or VU / USB)                | 5V Power Input (Powered via USB)           |
| ---                      | ---                      | ---                              | ---                                        |
|                          | DO (Digital Out)         | LEAVE EMPTY                      | Not Used                                   |
| ---                      | ---                      | ---                              | ---                                        |
|                          |                          |                                  |                                            |
| ---                      | ---                      | ---                              | ---                                        |
| **0.96" OLED Display**   | SCL                      | D1 (GPIO 5)                      | I2C Clock Signal                           |
| ---                      | ---                      | ---                              | ---                                        |
| _(Yellow-Blue Screen)_   | SDA                      | D2 (GPIO 4)                      | I2C Data Signal                            |
| ---                      | ---                      | ---                              | ---                                        |
|                          | GND                      | GND                              | Ground (-)                                 |
| ---                      | ---                      | ---                              | ---                                        |
|                          | VCC                      | 3V3                              | 3.3V Power Input                           |
| ---                      | ---                      | ---                              | ---                                        |
|                          |                          |                                  |                                            |
| ---                      | ---                      | ---                              | ---                                        |
| **3-Pin Passive Buzzer** | S (Signal)               | D7 (GPIO 13)                     | Frequency/Melody Output                    |
| ---                      | ---                      | ---                              | ---                                        |
|                          | VCC (or +)               | 3V3                              | 3.3V Power Input                           |
| ---                      | ---                      | ---                              | ---                                        |
|                          | GND (or -)               | GND                              | Ground (-)                                 |
| ---                      | ---                      | ---                              | ---                                        |
|                          |                          |                                  |                                            |
| ---                      | ---                      | ---                              | ---                                        |
| **Red Warning LED**      | \+ (Long Leg / Anode)    | D5 (GPIO 14)                     | Rhythmic Alert Light (Add a 220Ω resistor) |
| ---                      | ---                      | ---                              | ---                                        |
|                          | \- (Short Leg / Cathode) | GND                              | Ground (-)                                 |
| ---                      | ---                      | ---                              | ---                                        |
|                          |                          |                                  |                                            |
| ---                      | ---                      | ---                              | ---                                        |
| **Control Button**       | Pin 1                    | D6 (GPIO 12)                     | Interrupt Signal                           |
| ---                      | ---                      | ---                              | ---                                        |
|                          | Pin 2                    | GND                              | Chassis / Ground (-)                       |
| ---                      | ---                      | ---                              | ---                                        |

## **2\. Core Operational Logic and Algorithms**

### **A. Non-Blocking Time Management (millis() Based Architecture)**

The code architecture completely avoids bulky, processor-blocking delay() functions. All siren rhythms, screen saver timeouts, and button scans are managed asynchronously in the background using the processor's internal millisecond counter (millis()). This ensures that even if the gas threshold is breached at the exact millisecond a button is pressed, the system responds instantly with zero latency.

### **B. Smart Auto-Mute (Mute Reset) Safety Interlock**

If the button is pressed during an active alarm, the system immediately switches to MUTED... mode, instantly stopping the audible siren and the flashing LED.

Once the ambient smoke/gas level drops below the designated danger threshold (400), the mute interlock automatically resets. This critical feature ensures that the device remains fully alert against any new gas leaks that might occur hours or days later, eliminating the safety risk of a permanently silenced alarm.

### **C. Smart Screen Saver Mode (OLED Burn-In Protection)**

If the button is not pressed for 15 seconds while on the Graph or Log screens, the system automatically switches to the Screen Saver (Mode 0) to protect the OLED pixels and extend the display's lifespan. While in screen saver mode, pressing the button bypasses intermediate menus and returns the user directly to the Home Screen (Mode 1).

## **3\. Display Interface Modes (UI Menu)**

Under normal environmental conditions, the user can smoothly cycle through 3 distinct display modes using the button. The layout is optimized to match the hardware color boundaries of the dual-color screen:

- **Mode 0 (Screen Saver):** Displays the time in a clean HH:MM:SS format on the narrow top yellow strip, and the date in DD.MM.YYYY format on the lower blue strip. Air quality data is hidden.
- **Mode 1 (Home Screen):** The time remains stationary in the top yellow area, while the date and real-time air quality value (e.g., Air Quality: 120) are cleanly displayed in the lower blue area without overlapping.
- **Mode 2 (Real-Time Graph & Bar):** Schematically displays the air cleanliness using both text (AIR: CLEAN) and a visual progress bar.
- **Mode 3 (Persistent Alarm History):** Neatly lists the 3 most recent alarm logs read directly from the flash memory.

## **4\. Persistent File System and Log Structure (LittleFS)**

The SmokeNode does not store historical alarm records on RAM, meaning data is safely preserved even during a power outage or device shutdown.

- **Log Format:** HH:MM - DD-MM-YYYY (e.g., 17:58 - 15-02-2026)
- **Logging Algorithm:** The moment the gas value exceeds the 400 threshold, the system fetches the current atomic time from the internet (NTP) and appends it as a new line to the internal /alarm_log.txt file within the LittleFS file system.
- **Lock Mechanism:** To prevent unnecessary memory consumption while the gas level remains high, only a single log entry is recorded for the duration of the same alarm wave. A new log line is only appended after the gas clears completely and rises again.

## **5\. Firmware Upload and Initial Setup Instructions**

To compile and upload the project onto your ESP8266 (NodeMCU) board, please follow these software configuration steps in order:

### **1\. Arduino IDE Board Settings**

SmokeNode requires specific partitioning of the ESP8266 flash memory to operate stably and retain logs across power cycles.

**Defining the ESP8266 Board Library:**

- Open the Arduino IDE.
- Navigate to **File -> Preferences**.
- Paste the following URL into the **Additional Boards Manager URLs** field and click OK: \[<http://arduino.esp8266.com/stable/package_esp8266com_index.json\>](<http://arduino.esp8266.com/stable/package_esp8266com_index.json>)
- Click the **Boards Manager** icon on the left menu, type esp8266 in the search box, and click **Install**.

**Board, Flash Size, and Port Selection:**

- Connect your board to the computer using a USB cable.
- Go to **Tools -> Board -> ESP8266 Boards** and select **NodeMCU 1.0 (ESP-12E Module)**.
- Go to **Tools -> Flash Size** and strictly select **"4MB (FS:2MB)"**. _(This setting is mandatory for the LittleFS file system to allocate space for logging)._
- Go to **Tools -> Port** and select the active COM port your board is connected to.

### **2\. Installing Required Libraries**

To compile the code without errors, install the following libraries via the Arduino IDE **Library Manager**:

- **Adafruit SSD1306:** Required to drive the OLED display. Search for it in the library manager and install it. If prompted to install additional dependencies during installation, select **"Install All"** (this will automatically install _Adafruit GFX_).
- **LittleFS & ESP8266WiFi:** These libraries come pre-packaged with the ESP8266 board core library installed in the previous step; no external installation is required.

### **3\. Code Editing and Uploading**

- Open the primary .ino sketch file from the project directory using Arduino IDE.
- Locate the following lines at the very beginning of the code and update them with your wireless network credentials:

C++

const char\* ssid = "YOUR_WIFI_NAME";

const char\* password = "YOUR_WIFI_PASSWORD";

-   
   Click the **Upload** button (Right-pointing Arrow icon) in the top-left corner to flash the code to your board.

## **6\. Software Testing and Verification Protocol**

Once the upload is complete, you can verify the integrity and correct operation of the system using the following test scenarios:

- **Network & Time Sync Check:** When the device is first powered up, the screen will display "Connecting to Wi-Fi...". The moment the Wi-Fi connection is established, it pulls the atomic time from the NTP server, displays "SYSTEM READY!", and boots directly into Mode 1 (Home Screen).
- **OLED Burn-In Protection (Screen Saver):** If no button inputs are registered for 15 seconds while on the Home or Graph screens, the software automatically triggers Mode 0 (Screen Saver). In this mode, the time remains split-free in the upper yellow strip while the air quality data is hidden.
- **Interrupt Latency Test:** Pressing the button while the device is in screen saver mode will instantly bypass intermediate menus and snap back to Mode 1 within milliseconds.
- **Persistent Logging Verification:** When the sensor reading crosses the 400 threshold, the software generates or opens the /alarm_log.txt file in the internal flash memory and logs the timestamp. Navigate to Mode 3 (Alarm History) using the button to verify the last 3 records are listed correctly.
- **Smart Mute Logic:** Pressing the button while an alarm is actively sounding will silence the buzzer and display MUTED... on the screen. As soon as the gas value drops below 400, the software automatically clears the mute lock, re-arming the system for the next emergency.
