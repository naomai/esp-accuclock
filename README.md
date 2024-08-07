# AccuClock-ESP

Atomic time accuracy for your wall clock.

Many of modern devices use a special signal for their time keeping. The signal comes from a small component, called **crystal oscillator**, which is vibrating at a very high rate. *In a perfect world, it should be exactly 32768 times a second.*

Unfortunately, that rate might be varying. The temperature of the device, and imperfections in manufacturing might contribute to small drifts. *In the scale of week, your clock could be off by few seconds.*

This project aims to create a more stable replacement for the resonating crystals. As a reference for our internal clock, we're going to use atomic time, publicly available on the internet - via **NTP protocol**. The hardware part is a **ESP8266** board, which is perfect for this task - it has WiFi capability, and is *very cheap*.

## Several things to consider

* ESP drives its pins with 3.3 volts. This might be too much for the circuitry of the clocks, which expects weak sine signal for amplification.
* ESP provides a library for generating waveforms, which are not disturbed by other tasks ran by RTOS.
  * The frequency seems to be stable, but its generation is tied to CPU cycles. This means that in khz range, we can only tune to frequencies with 20-ish Hz accuracy.
    ```
    System clock is 80MHz
    80000000/32768 = 2441.40625 cpu cycles per one wave cycle
    ```
  * We should dynamically switch between two closest freqencies, while keeping track of how many wave cycles were lost/overran.
    ```
    2441 ccl = 32773 Hz
    2442 ccl = 32760 Hz
    ```
  * Our aim is not to have the exact frequency at all times, but to provide a signal which, on average, has the same number of cycles over a longer period of time.
  * The internal timekeeping should be based solely on CPU clock. I cannot find clear information whether ESP uses additional RTC clock, or bases its `time` library on CPU clock. My first implementation shows drift that is significant, regular, but too small to be related to waveform generation. 

    Hints: [1](https://www.esp8266.com/viewtopic.php?p=10180)
* For the best accuracy, use NTP library with millisecond sync ablility. [ESPNtpClient](https://github.com/gmag11/ESPNtpClient)
* Adjusting the clock after sync is done by slightly slowing down/speeding up the frequency over some period (30s)
* To save power, disable WiFi radio after sync - need to further investigate the behavior of ESPNtpClient.
* Create a test program for second ESP, which will be used for measuring the stability.

There is a similar project for analog clocks - [ESP8266-WiFi-Analog-Clock](https://github.com/jim11662418/ESP8266-WiFi-Analog-Clock). The program skips the oscillator part by completely replacing the electronics inside clock - manually controls the movement of second hand.
  

