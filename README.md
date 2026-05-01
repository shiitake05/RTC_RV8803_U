# Micro Crystal RV8803
This library is part of an idea to create a unified API for handling RTCs, and is a library (device driver) for the [Micro Crystal RV8803] [RV8803].

## Operation Verification

| CPU | Model | Compatibility |
|---|---|---|
| AVR | Arduino Mega | ○ |
| SAMD | Arduino MKR WiFi1010 | ○ |
| SAM | Arduino Due | ○ |
| ESP32 | Switch Science ESP developer32 | ○ |

- Board used for operation verification: [RV-8803-BOARD](https://www.marutsu.co.jp/pc/i/1556263/)

## External Links

- Data sheet - [https://tamadevice.co.jp/pdf/mc/rtc/jpn/rv-8803-c7-datasheet-j.pdf](https://tamadevice.co.jp/pdf/mc/rtc/jpn/rv-8803-c7-datasheet-j.pdf)
- Application Manual - [https://tamadevice.co.jp/pdf/mc/rtc/RV-8803-C7_App-Manual_ja_1.pdf](https://tamadevice.co.jp/pdf/mc/rtc/RV-8803-C7_App-Manual_ja_1.pdf)
- Product Information - [https://www.microcrystal.com/jp/%E8%A3%BD%E5%93%81/%E3%83%AA%E3%82%A2%E3%83%AB%E3%82%BF%E3%82%A4%E3%83%A0%E3%82%AF%E3%83%AD%E3%83%83%E3%82%AF%E3%83%A2%E3%82%B8%E3%83%A5%E3%83%BC%E3%83%AB/rv-8803-c7/](https://www.microcrystal.com/jp/%E8%A3%BD%E5%93%81/%E3%83%AA%E3%82%A2%E3%83%AB%E3%82%BF%E3%82%A4%E3%83%A0%E3%82%AF%E3%83%AD%E3%83%83%E3%82%AF%E3%83%A2%E3%82%B8%E3%83%A5%E3%83%BC%E3%83%AB/rv-8803-c7/)


## Notes on Use

The ``RTC_RV8803_U.h`` contains flags that enable features for debugging and performance testing.

Please enable or disable them as needed.

```
#define DEBUG
```
## Sample Program
The sample program is designed to verify whether the RTC registers are properly set when each function of this driver is executed.

Please compile and install after enabling the ``DEBUG`` definition in ``RTC_RV8803_U.h``.

Disabling the following line will enable detailed messages and registration content verification.
```
#undef DEBUG
```

Enabling the following line will dump the contents of all registers after each test has rewritten the registers.
```
#define DUMP_REGISTER // Enable this if you want to see a dump of the register values ​​after rewriting the register values ​​(also enable DEBUG)
```
# API
To understand the following functions, you need to know the meaning of the bits in each RTC register, so please read while referring to the [Datasheet] [Datasheet].

## Initialization Related
### Object Creation
```
RTC_RV8803_U(TwoWire * theWire, int32_t rtcID=-1)
```
Creates an object by specifying the I2C interface and ID used by the RTC.

### Initialization
```
bool begin(bool init=true, uint8_t addr=RTC_RV8803_DEFAULT_ADRS)
```
Initials the RTC. Providing ``false`` as an argument will only perform minimal interface initialization without setting the time, which is useful when restarting the Arduino after setting the time on the RTC.
The second argument should be specified if you do not want to use the RTC's default I2C address.

| Return Value | Meaning |
|---|---|
|true|Initialization Successful|
|false|Initialization Failed|

## Obtaining RTC Information
Member function to obtain information about the RTC chip type and function.
```
void getRtcInfo(rtc_u_info_t *info)
```

## Time Information Related
### Setting Time
```
bool setTime(date_t* time)
```
Sets the time given as an argument to the RTC.
| Return Value | Meaning |
|---|---|
|true|Setting Successful|
|false|Setting Failed|

### Obtaining Time
```
bool getTime(date_t* time)
```
Stores the time information obtained from the RTC in the structure given as an argument.

| Return Value | Meaning |
|---|---|
|true| Successful Retrieval|
|false| Failed Retrieval|

## Power Supply Related
This RTC has a function to detect power supply voltage drops, and this data is stored in the lower 2 bits (0 and 1) of the flag register (0x0E). This 2-bit data can be accessed using the following functions:

### Obtaining Power Supply Voltage Drop Information
```
int checkLowPower(void)
```
Reads and returns the lower 2 bits of the flag register (0x0E). A negative return value indicates an error.

### Clearing Power Supply Voltage Drop Information
```
int clearPowerFlag(void)
```
Sets both lower 2 bits of the flag register (0x0E) to 0.

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Setup successful |
|RTC_U_FAILURE | Setup failed |
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |

## Clock Signal Output
Since only one type of signal can be controlled, the value of the first argument ``num'' of the following function is limited to 0.


### Signal Output ON/OFF
```
int controlClockOut(uint8_t num, uint8_t mode)
```
|Value of ``mode`` | Action|
|---|---|
| 0 | Stop signal output |
| 1 | Start signal output |

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Setting successful|
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc.|

### Signal Frequency Setting
```
int setClockOutMode(uint8_t num, uint8_t freq)
```

The signal frequency is determined by the combination of the values ​​of bits 2 and 3 (FD) of the extended register (0x0D). Since the value of freq is overwritten on these 2 bits, freq takes a value between 0 and 3.


| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Setup successful |
|RTC_U_FAILURE | Setup failed |
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |

### Signal Output Setting
```
int setClockOut(uint8_t num, uint8_t freq, int8_t pin)
```
The pin number given in the third argument is stored internally, and ``setClockOutMode(num, freq)`` and ``controlClockOut(num,1)`` are executed sequentially.

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Setup successful |
|RTC_U_FAILURE | Setup failed |
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |

## Controlling the Clock's Progression
This RTC has two functions to adjust the clock's progress: ``setOscillator()`` which adjusts the internal clock's progress, and ``controlClock()`` which resets the internal counter for seconds to 0.

### Setting Parameters to Control the Clock's Progression
```
int setOscillator(uint8_t mode)
```
The value of mode is written directly to the offset register (0x2C). Refer to the datasheet for the meaning of mode.


| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Setup successful |
|RTC_U_FAILURE | Setup failed |
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |

### Referencing the value of the parameter that controls the clock speed
```
int getOscillator(void)
```
Reads and returns the value of mode in the offset register (0x2C). If it is 0 or greater, it is the value of the offset register; if it is negative, it is an error.

### Reset to less than a second
```
int controlClock(void)
```
Writes 1 to the least significant bit (RESET) of the control register (0x0F), resetting the less-than-second portion of the internal counter to 0.

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Setting successful |
|RTC_U_FAILURE | Setting failed |
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |

## Interrupts
In this RTC, interrupt information other than power supply-related information is stored in bits 2 to 5 of the flag register (0x0E).

### Get Interrupt Information
```
int checkInterupt(void)
```

| Return Value | Meaning |
|---|---|
| 0 or greater | Value of bits 2 to 5 of the flag register (0x0E) |
|RTC_U_FAILURE | Read failed |

### Clear Interrupt Information
```
int clearInterupt(uint16_t type)
```
Sets the bit specified by type (the bit that is always 1) in bits 2 to 5 of the flag register (0x0E) to 0.

For example, to set two bits of the flag register to 0, set the least significant bit of type to 1.

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Setting successful |
|RTC_U_FAILURE | Setting failed |
|RTC_U_ILLEGAL_PARAM | Setting an unsupported parameter, etc. |

## Alarm
Since this RTC has only one type of alarm, the value of the first argument ``num`` of the following function is limited to 0.

### Alarm Setting
```
int setAlarm(uint8_t num, alarm_mode_t * mode, date_t* timing)
```
In this RTC, you can specify the time the alarm fires in minutes, hours, and days (or days of the week), but you cannot specify both the day and day of the week simultaneously. Which one to use is specified by the ``type`` member of ``mode``. Furthermore, the ``useInteruptPin'' parameter in ``mode'' determines whether or not to output the interrupt signal externally.

Specify using ``.

|mode structure member|Write destination|Meaning |
|---|---|---|
|useInteruptPin|Control register (0x0F) AIE (3 bits)|Specifies whether to send an interrupt signal (1) or not (0) to the INT pin when the alarm is triggered|
|type|Extended register (0x0D) WADA (6 bits)|Specifies whether the alarm is specified as a day (1) or a day of the week (0)|

The actual time the alarm is triggered is specified by ``timing'', but please set any unused items from the minute, hour, and day (or day of the week) to 0xFF.

For example, setting ``timing'' as follows will trigger the alarm at 1 minute past every hour.

```
timing.minute = 1;
timing.hour = 0xFF;
timing.wday = 0xFF;
timing.mday = 0xFF;
```
| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Setting successful |
|RTC_U_FAILURE | Setting failed |
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |

### Switching the external output of the alarm interrupt signal
```
int setAlarmMode(uint8_t num, alarm_mode_t * mode)
```
This function changes only the ``useInteruptPin` setting among the items specified in ``mode`` of ``setAlarm()``.

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_FAILURE | Configuration failed |
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |

### Alarm ON/OFF
```
int controlAlarm(uint8_t num, uint8_t action)
```
Turns the alarm ON/OFF according to the argument action.

|action value|Meaning|
|---|---|
|0|Stop alarm|
|1|Restart alarm (only for alarms that have been stopped)|

|Return Value|Meaning|
|---|---|
|RTC_U_SUCCESS|Setting successful|
|RTC_U_FAILURE|Setting failed|
|RTC_U_ILLEGAL_PARAM|Setting of unsupported parameters, etc.|

## Timer
This RTC has two types of timer functions: a periodic timer that generates an interrupt every second (or every minute), and a normal timer function. The following function controls the periodic timer when the first argument ``num'' is 0, and the normal timer when ``num'' is 1.

### Timer Setup and Activation
```
int setTimer(uint8_t num, rtc_timer_mode_t * mode, uint16_t multi)
```
| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Setup successful|
|RTC_U_FAILURE | Setup failed|
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc.|

#### Operation when ``num=0``

Since the periodic timer fires every minute or every second, the ``multi`` argument is ignored.
`mode`` is used as follows:

|Member|Purpose|Content|
|---|---|---|
|uint8_t pulse | Unused ||
|uint8_t repeat | Unused ||
|uint8_t useInteruptPin | Assigned to UIE (5-bit) of the control register (0x0F) | Whether or not to send the interrupt signal externally |
|uint8_t interval | Assigned to USEL (5-bit) of the extension register (0x0D) | 0: Every second, 1: Every minute |

#### Operation when ``num=1``

When ``num=1``, it becomes a normal countdown timer, so the argument ``multi`` is processed as follows.

|multi bit| processing|
|---|---|
|12 to 15| ignored|
|8 to 11| assigned to Timer Counter 1 register (0x0C)|
|0 to 7| assigned to Timer Counter 0 register (0x0B)|

After processing ``multi'', ``setTimerMode(1,mode)'' and ``controlTimer(1,1)'''' are processed in that order.

### Timer Parameter Change
```
int setTimerMode(uint8_t num, rtc_timer_mode_t * mode)
```
| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Setting successful |
|RTC_U_FAILURE | Setting failed |
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |

#### Behavior when ``num=0``

``mode`` is used as follows:

|Member|Purpose|Content|
|---|---|---|
|uint8_t pulse | Unused ||
|uint8_t repeat | Unused ||
|uint8_t useInteruptPin | Assigned to UIE (5-bit) of the control register (0x0F) | Whether or not to send the interrupt signal externally |
|uint8_t interval | Assigned to USEL (5-bit) of the extension register (0x0D) | 0: Every second, 1: Every minute |

#### Operation when ``num=1``

``mode`` is used as follows:


|Members|Purpose|Content|
|---|---|---|
|uint8_t pulse | Unused ||
|uint8_t repeat | Unused ||
|uint8_t useInteruptPin | Assigned to TIE (4-bit) of the control register (0x0F) | Whether or not to output the interrupt signal externally |
|uint8_t interval | Assigned to TD (0,1-bit) of the extension register (0x0D) | Selection of the reference frequency for the countdown timer |

### Timer On/OFF

```
int controlTimer(uint8_t num, uint8_t action)
```

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Setting successful |
|RTC_U_FAILURE | Setting failed |
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |

|Value of ``mode`` | Operation |
|---|---|
| 0 | Stop Timer |
| 1 | Start Timer Operation |

## External Events
This RTC allows you to take a snapshot of a specific point in time in seconds and 10ms increments by adjusting the voltage applied to a particular terminal from an external source. This is called an "external event."

### Setting for External Events
```
int setEvent(event_mode_t *mode)
```
|Members of mode|Value Usage|
|---|---|
|uint8_t useInteruptPin|EIE (2 bits) of the control register (0x0F)|
|bool capture|ECP (7 bits) of the event setting register (0x2F)|
|uint8_t level|EHL (6 bits) of the event setting register (0x2F)|
|uint8_t filter|ET (4,5 bits) of the event setting register (0x2F)|
|bool reset|ERST (0 bits) of the event setting register (0x2F)|

|Return Value|Meaning|
|---|---|
|RTC_U_SUCCESS|Setting Successful|
|RTC_U_FAILURE|Setting Failed|
|RTC_U_ILLEGAL_PARAM |Unsupported Parameter Settings, etc.|

### Obtaining Event Information
```
int getEvent(void)
```
| Return Value | Meaning |
|---|---|
|0 or greater | Acquired snapshot data (explained below) |
|RTC_U_FAILURE | Read failure |
|RTC_U_NO_EXTERNAL_EVENT | No external event occurred |

| Each bit of the return value | Meaning |
|---|---|
|16 to 31 | Unused |
|8 to 15 | Second data (register number 0x21) |
|0 to 7 | Time in 10ms units (register number 0x20) |

## Accessing the SRAM Area
In the RV8803, register number ``0x07`` can be used as the SRAM area. This register is used when the first argument ``addr`` of the following two functions is 0.
Also, ``len`` can only be 1.

### Reading from SRAM
```
int getSRAM(uint8_t addr, uint8_t *array, uint16_t len)
```
The data at register number ``0x07'' is assigned to the first element of the array ``array''.

### Writing to SRAM
```
int setSRAM(uint8_t addr, uint8_t *array, uint16_t len)
```
The data at the first element of the array ``array'' is written to register number ``0x07''.

[データシート]:https://tamadevice.co.jp/pdf/mc/rtc/jpn/rv-8803-c7-datasheet-j.pdf
[アプリケーションマニュアル]:https://tamadevice.co.jp/pdf/mc/rtc/RV-8803-C7_App-Manual_ja_1.pdf
[RV-8803-BOARD]:https://www.marutsu.co.jp/pc/i/1556263/
[RV8803]:https://www.microcrystal.com/jp/%E8%A3%BD%E5%93%81/%E3%83%AA%E3%82%A2%E3%83%AB%E3%82%BF%E3%82%A4%E3%83%A0%E3%82%AF%E3%83%AD%E3%83%83%E3%82%AF%E3%83%A2%E3%82%B8%E3%83%A5%E3%83%BC%E3%83%AB/rv-8803-c7/


[github]:https://github.com/sparkfun/SparkFun_DS3231_RTC_Arduino_Library
[Grove]:https://www.seeedstudio.io/category/Grove-c-1003.html
[SeedStudio]:https://www.seeedstudio.io/
[AdafruitUSD]:https://github.com/adafruit/Adafruit_Sensor
[shield]:https://www.seeedstudio.com/Base-Shield-V2-p-1378.html
[M0Pro]:https://store.arduino.cc/usa/arduino-m0-pro
[Due]:https://store.arduino.cc/usa/arduino-due
[Uno]:https://store.arduino.cc/usa/arduino-uno-rev3
[UnoWiFi]:https://store.arduino.cc/usa/arduino-uno-wifi-rev2
[Mega]:https://store.arduino.cc/usa/arduino-mega-2560-rev3
[LeonardoEth]:https://store.arduino.cc/usa/arduino-leonardo-eth
[ProMini]:https://www.sparkfun.com/products/11114
[ESPrDev]:https://www.switch-science.com/catalog/2652/
[ESPrDevShield]:https://www.switch-science.com/catalog/2811/
[ESPrOne]:https://www.switch-science.com/catalog/2620/
[ESPrOne32]:https://www.switch-science.com/catalog/3555/
[Grove]:https://www.seeedstudio.io/category/Grove-c-1003.html
[SeedStudio]:https://www.seeedstudio.io/
[Arduino]:http://https://www.arduino.cc/
[Sparkfun]:https://www.sparkfun.com/
[SwitchScience]:https://www.switch-science.com/



