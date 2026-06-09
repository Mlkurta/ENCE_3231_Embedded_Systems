
# Balancing Robot


## Introduction

This project is a two-wheel self-balancing robot built around an STM32 microcontroller and an MPU6050 inertial measurement unit. The robot uses real-time sensor fusion and closed-loop control to maintain an upright position while responding to user commands for forward, reverse, and turning motion.

The primary objective of the project is to explore the fundamentals of feedback control systems, embedded software development, and robotics. The balancing algorithm combines accelerometer and gyroscope measurements to estimate robot orientation, while a PID-based control loop continuously adjusts motor output to maintain stability.

This project serves as a platform for learning embedded systems, sensor integration, control theory, PCB design, and robotic system development. Future enhancements may include improved state estimation, autonomous navigation, and advanced motion control techniques.

## Design Phase

When it comes to designing embedded systems, it's a good idea to draw out what resources will be used or needed. Below is a system diagram used for the design.

System Diagram:

<img width="838" height="1077" alt="IMG_0647" src="https://github.com/user-attachments/assets/56d87b8b-5369-4933-bbff-ec2f2eff1e4c" />

Some key features include:

-Real-time pitch estimation using IMU data

-Closed-loop balance control using PID algorithms

-Differential drive motor control for steering and movement

-Wireless command interface via ESP8266 Wi-Fi module

-Custom-designed STM32 control PCB

-Modular software architecture for future expansion

-RGB indicator LED, useful for debugging.

Pin Planning:

<img width="590" height="550" alt="Screenshot 2026-06-07 201333" src="https://github.com/user-attachments/assets/e614a03c-0016-47da-a7bb-72df90bb5e4f" />

We would need peripherals for the following:

-I2C for the IMU (I2C1)

-Timer 4 channels 1 and 2 for encoder-based inverted pendulum control (if we got to it)

-SWDIO and SWCLK pins for programming and debugging

-USB ESD pins, since we will be programming via USB as default

-USART TX/RX for communication with the ESP8266 commands / telemetry (USART1)

-I2C for the hall-effect encoder needed to measure wheel speed (I2C3)

-A timer to generate regular interrupts for either hall-effect encoder I2C updates OR to measure PWM duty cycle using the OUT pin instead of I2C

-A GPIO for the LED (to indicate running status)

-Five more GPIO pins for the H-Bridge driver signals (AIN 1&2, BIN 1&2) and Standby

-Two timer-driver GPIO pins to set the PWM out for the H-Bridge (Timer 2, channels 1 and 2)

-A reset pin (NRST).

-Two RCC pins, since we'll be atttaching a 16 MHz external clock

-A Timer controlled GPIO for setting colors on the WS2812 RGB LED. This is useful for knowning if the robot received commands from the controller.

-One last INT GPIO to trigger an interrupt if certain motion-related events happen



## Prototype Phase


## PCB Design

<img width="1205" height="804" alt="image" src="https://github.com/user-attachments/assets/9ea69e72-081f-49be-bcbd-0ee2c5b34d3d" />

One error I made is I did not use a separate name for the 5V voltage regulator; I used the same LDR for 3.3V and 5V. Fortunately the footprins are the same, and the correct parts were available when soldering.

<img width="667" height="524" alt="Board_Back" src="https://github.com/user-attachments/assets/b4170aba-498d-4131-9dd5-7aeb472aae65" />

<img width="667" height="524" alt="Board_Front" src="https://github.com/user-attachments/assets/0c4f34c0-bb19-47da-8e32-1fbde7892ca5" />

<img width="726" height="629" alt="Board3D" src="https://github.com/user-attachments/assets/97852e91-7424-4f01-a811-9dc0087fa7eb" />

I was intentional about flipping the ESP8266 so that the package would be more compact. In hindsight, it mightve been a better idea to not do that as now the antenna radiates over the PCB; generally not a good design practice.

## Assemble Stage

This was, for the most part, a hand-soldered project. 

<img width="1008" height="756" alt="IMG_8146" src="https://github.com/user-attachments/assets/3d16b5a9-2bec-4b3a-a9d2-954b41d29b84" />

Because I did not separately distinguish between my two LDOs, I ended up soldering the 3.3V LDO to the 5V position; so I had to undo that. 

Additionally, I bridged some pins on the MCU, though I got rid of them. Or so I thought. 

## 3.3V Rail short to GND

Once all of the components were soldered to the PCB, I found out I hard a short between 3.3V and GND. After probing around, applying small currents and looking through an infrared camera, and resoldering suspect areas, I still could not find the short.

I found an online technique for locating shorts on a PCB by injecting a constant current into the affected power rail (3.3 V and GND in this case). Using a bench multimeter capable of measuring very small voltage differences, the millivolt drop between the 3.3 V rail and ground was measured at various locations across the board. A handheld multimeter would likely not provide sufficient resolution for this method. The principle is that locations electrically closer to the short will exhibit a lower voltage drop.

With a current of 500 mA injected through the debug header, measurements were taken at nearly every accessible 3.3 V node on the PCB. Most locations measured approximately 7 mV, while capacitor C5 and the lower-right 3.3 V pin of the STM32 measured approximately 5 mV. This suggested that the short was located near the microcontroller. Upon closer inspection under a microscope, a small solder bridge was discovered between adjacent 3.3 V and ground pins/vias. The bridge was partially hidden behind the pins, making it difficult to detect during normal visual inspection.

Once the short was removed, the problem was solved.

## H-Bridge Issues

I found out that the motor test program worked, but not for one direction for one motor connector. 

I had trouble deciphering what was going on. I thought I had the idea nailed down until I realized I didnt. I decided to make things a little more concrete and make a psuedo truth table:

<img width="834" height="1076" alt="IMG_0648" src="https://github.com/user-attachments/assets/1a3c421c-db25-49cb-b687-5d64961ff9ee" />

This meant that the A01 output of the TB6612 was not ouputting a PWM signal. Using an oscilloscope, I conneced the A01 connector to an oscilloscope, and used the other probe to check standby, PWM and AIN1 pins. 

The PWM and AIN1 were all what they should be: PWM doing PWM, and AIN1 was at 0V.  From this, I began to think the connection between the STM32 and motor controller was bad, so I checked and there was good continuity between the two pins. I probed with the scope again and now I saw 3.3V. I thought for sure the motor controller was faulty or burnt out somehow with the information. I probed again and now noticed the A01 show PWM for a split second. 

Then it dawned on me. If the pin (AIN1) was not a perfect solder, all of this could be explainable. Sometimes showing 3.3V when it should, sometimes not. Even when it was 3.3V, A01 was still flat. When pressing again (occasionally), A01 output PWM like it should.  I reflowed the AIN1 pin, and immediately the motor problem was fixed.

## 3 Wheel Robot

It was now time to test the robot. With all of the motors tested, an acrylic platform was drafted and made with a laser cutter.

<img width="610" height="337" alt="image" src="https://github.com/user-attachments/assets/28b9ac25-c310-44aa-ac3f-2195afa4e953" />

Then the body was assembled with everything connected, except the hall effect wheel encoder. 

Connecting to the websocket, I could now view the telemetry. All but the speed, anyways as the encoder wasn't installed/configured.

<img width="295" height="639" alt="IMG_8338" src="https://github.com/user-attachments/assets/5a684fe6-3117-4e8a-a8de-2fed57b2c9a5" />

The robot could now be controlled with commands from my phone. I should note tha originally, the telemetry data and commands did not work. The ESP8266 would blink when commands were sent, but nothing would register. This is where the RGB LED was helpful.

Scouring through the code, the timer 3 interrupt was performing a TimerCompleteCallback() function, where the encoder read was triggered. The encoder read contains an I2C_MemRead() function, which has a timeout of 100ms.  This interrupt interval was also 100ms.  The UART is triggered through polling in the main while(1) loop.  So the UART read and writes never executed because every 1/10th of a second, an interrupt triggered, spending 1/10th of a second looking for an I2C device which wasn't there. The fix was either disabling or commenting out Timer 3.

Baby steps.

<img width="421" height="750" alt="3Wheel" src="https://github.com/user-attachments/assets/575b9c0e-88a8-4ebb-b091-b9bb3c427400" />

Clearly there was a difference in motor speed, but otherwise we were about ready to start programming and take off the training wheel(s).

## Encoder Problems

The encoder originally seemed like it was reading values, but after lots of printing different values to the "Speed" variable in the websocket, I found out that this originated from a raw value of 128. 

I ended up using a command: HAL_StatusTypeDef status = HAL_I2C_MemRead();  and then printing status to the websocket.  I found out that this was 1, which I thought was good, but 1 means ERROR.

I used a device scanner function to determine what device IDs would respond, and then I found out that it reponds to:


Device ID

<img width="295" height="639" alt="IMG_8339" src="https://github.com/user-attachments/assets/c4efb273-a7b9-4eac-8fbb-25913529fc98" />

This showed a device ID (decimal = 54, which is Hex 0x36).

Then I repeated the above step with determining the I2C connection (Speed = 0, which I printed the HAL_StatusTypeDef status to (casted as int)):

HAL == OK

<img width="295" height="639" alt="IMG_8340" src="https://github.com/user-attachments/assets/3e776125-f507-4518-93e8-705dff1c63ef" />

I still wondered why I couldnt get any speed values from this device. The register addresses were correct. Then I tried connecting it to a Teensy and using Arduino and ChatGPT to detemine if that worked. 

All I was able tto determine was that the device was acknowledging the correct device address, but nothing after attempts to read the registers. I ended up looking at the IC on the board and there was an interesting observation:

<img width="845" height="638" alt="IMG_8341" src="https://github.com/user-attachments/assets/4d985973-521e-4568-90d0-05c8bc0002c7" />


This was an AS5600. Upon looking at the datasheet, 0x32 is exactly the device address, but the read registers were different. In order to get raw angle, you must read 2 bytes beginning at 0x0C for angle. Then the degree conversion was different. Once that was determined, I had a means to measure angle and speed. I thought this was a good learning experience, as occasionally a part will be substituted with an equivalent (in a company, or group project).  Knowing to use a I2C scanning function can be helpful in going back to square one. Or, if in doubt, look at the IC markings.


Balancing the Wheel velocities.


<img width="381" height="286" alt="image" src="https://github.com/user-attachments/assets/3240d63b-e6a9-4b09-9566-7023d75f9801" />


To characterize the drive motors and improve speed matching between the left and right wheels, a wheel speed calibration procedure was performed. The robot was elevated so the wheels could rotate freely, and PWM duty cycle values were applied in increments of 10 throughout the operating range. For each PWM setting, a video recording was taken while the wheel completed five full rotations. The recorded footage was then reviewed to determine the elapsed time required for the five rotations at each PWM value.

The measured rotation times were converted into wheel speed data and used to generate a relationship between PWM command and rotational speed. This calibration revealed variations in motor performance across the operating range and provided a basis for compensating differences between the two drive motors. By using the experimentally measured speed data, more consistent wheel velocities could be achieved, improving straight-line tracking and overall balancing performance of the robot.

## Control Tuning


The robot uses a PD balancing controller based on fused accelerometer and gyroscope measurements. Wheel velocity feedback is incorporated to improve translational control and reduce steady-state drift, allowing the robot to maintain balance while tracking commanded forward and reverse speeds.

$u(t) = K_p(\theta_{target} - \theta) - K_d\dot{\theta} + K_v(v_{target} - v)$

A complementary filter was used to estimate robot pitch by combining gyroscope and accelerometer measurements. The gyroscope provides accurate short-term angular rate information but is susceptible to drift when integrated over time. The accelerometer provides an absolute estimate of tilt relative to gravity but is sensitive to vibration and noise. The complementary filter blends these measurements, using a weighting factor α to combine the short-term stability of the gyroscope with the long-term stability of the accelerometer.

$\theta_k = \alpha \left(\theta_{k-1} + \omega_k \Delta t\right) + (1-\alpha)\theta_{acc,k}$

An α value of 0.995 was selected, resulting in approximately 98% reliance on the integrated gyroscope measurement and 2% reliance on the accelerometer measurement during each update cycle.

