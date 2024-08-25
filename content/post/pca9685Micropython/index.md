+++
author = "Hussam Al-Hertani"
title = "MicroPython Library for the PCA9685"
date = "2024-08-23"
draft = false
description = "PCA9685 driver in Micropython"
[params]
  math = true
tags = [
    "Micropython", "ESP32", "Electronics"
]
categories = [
    "Micropython", "ESP32", "Electronics"
]
+++

The PCA9685 chip is a 16 channel PWM controller with 12-bits of resolution. It can control via PWM up to 16 LEDs. Thanks to the fact that it can output a variable frequency from 1526Hz down to 24Hz, it can also be used to control up to 16 servos. Servos usually expect a 50Hz signal (20ms period). The position of the shaft is a function of the ON time. For many standard servos: 

- 1ms ON time moves the shaft to the -90 degree position. 
- 2ms ON time moves the shaft to the +90 degree position. 
- 1.5ms ON time moves the shaft to the 0 degree position.

{{< figure src="fig1.png" caption="Figure 1. Servo ON time to shaft position mapping" width=500px >}}

These  ON time specifications are not always followed. The beefier MG996R servos for example, have a different set of ON times to shaft position mapping:

- 0.5ms ON time moves the shaft to the -90 degree position. 
- 2.5ms ON time moves the shaft to the +90 degree position. 
- 1.5ms ON time moves the shaft to the 0 degree position.

The PCA9685 chip can be controlled from a microcontroller over the I2C bus. Each one of its 16 channels can be configured to have a different duty cycle. All 16 channels however must have the same frequency; which is configured by writing a value into the PRESCALE register. This value divides the 25MHz internal oscillator frequency, into a PWM frequency between 24Hz and 1526Hz.

The duty cycle for each channel is configured via 2 pairs of registers. One pair determines when the ON pulse starts and the other pair determines when it ends.

I recently developed an Arduino shield for the PCA9685 I2C chip in KiCAD as a part of an educational board ecosystem that I call the **IOT Innovation Platform**. I will introduce these boards in future blog entries.

{{< figure src="fig2.jpeg" caption="Figure 2. My PCA9685 Arduino shield / daughterboard plugged into the IOT Innovation Platform" width=500px >}}

I decided to use an ESP32S3 microcontroller (also a part of the **IOT Innovation Platform** ) running Micropython for testing. So I had to write a PCA9685 Library in MicroPython. 

The PCA9685 Driver class provides low level functions that can set the PWM frequency for the PCA9685 as well as independently set the duty cycle for each one of the 16 channels either as a percentage, an ON time pulse duration, or via start and end count values for the ON Pulse. 

```Python
from pca9685 import PCA9685Driver
import time

# Initialize PCA9685 with default I2C settings
pwm = PCA9685Driver()

# Print I2C address of the device (should be 0x40)
print(pwm.i2c.scan())

# Set PWM frequency to 1200Hz and duty cycle to 60% on channel 1
pwm.set_pwm_frequency(1200)
pwm.set_pwm_dc_percent(1, 60)
time.sleep(5)
```


It also has higher level abstraction functions for servo control.

```Python
from pca9685 import PCA9685Driver
import time

# Initialize PCA9685 with default I2C settings
pwm = PCA9685Driver()

# Print I2C address of the device (should be 0x40)
print(pwm.i2c.scan())
# For channels 1, 14, and 15, vary the on-time from 0.5ms to 2.5ms (ideal for 180° rotation for MG996R servos)

# Set PWM frequency to 50Hz for servo control
pwm.set_pwm_frequency(50)

while True:
    for angle in range(0, 190, 10):
        pwm.servo_set_angle_custom(1, angle, 0.5, 2.5)  # channel 1
        pwm.servo_set_angle_custom(14, angle, 0.5, 2.5) # channel 14
        pwm.servo_set_angle_custom(15, angle, 0.5, 2.5) # channel 15
        print(f"angle: {angle}")
        time.sleep(0.1)
    for angle in range(180, -10, -10):
        pwm.servo_set_angle_custom(1, angle, 0.5, 2.5)
        pwm.servo_set_angle_custom(14, angle, 0.5, 2.5)
        pwm.servo_set_angle_custom(15, angle, 0.5, 2.5)
        print(f"angle: {angle}")
        time.sleep(0.1)
```
The complete PCA9685 driver class and demo code can be found on [my github page](https://github.com/halherta/pca9685). 