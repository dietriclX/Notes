# ESP32-CAM

ESP32-CAM-MB WiFi Bluetooth Development Board OV2640 Camera Module Micro USB Interface CH340G USB to Serial Interface Pack of 5

Consists of the "ESP32-CAM-MB" board, with the Micro-USB connector and the `RST` button (yes, only one button), and piggyback mounted the "ESP32-CAM" board.
"ESP32-S" & "WIFI+BT SoC Inside" marking on the Espressif-Module.
Markings on the chips not readable.

=== Output of "Arduino/ESP32/Identify" ===
```code
Chip is ESP32-D0WDQ6 Rev 1
Features: Dual Core, Wifi-BGN, BLE, Bluetooth
SPI RAM size: 4192107 Byte
Flash chip size: 4 MB
Flash chip frequency: 40 MHz
```

## GPIO0

As this ESP32CAM module has no dedicated button for GIO0, this pin is controlled by USB-TTL chip. I didn't know if it was `DTR` and/or `RTS`. Even though, the upload of a sketch was working, but I was unable to receive/see Serial.println(...) log message of my application.

Using GTKTerm, I was able to identify the right configuration (monitor_speed = used in application).

=== platformio.ini ===
```code
:
[env:esp32cam]
platform = espressif32
board = esp32cam
framework = arduino
monitor_dtr = 0
monitor_rts = 0
monitor_speed = 115200
:
```

For completeness the test cases with GTKTerm.

1. ensure `DTR` and `RTS` are set correctly
2. press `RST` button
3. watch output

Observed result:
| Baudrate | DTR | RTS | Output |
| --- | --- | --- | --- |
| 115200 | 0 | 0 | Ok, messages from application |
| 115200 | 0 | 1 | nothing |
| 115200 | 1 | 0 | Ok, ready to upload sketch |
| 115200 | 1 | 1 | nothing |

(* Output, indicating ESP32 ready to receive upload of new application.
```code
rst:0x1 (POWERON_RESET),boot:0x3 (DOWNLOAD_BOOT(UART0/UART1/SDIO_REI_REO_V2))
waiting for download
```
