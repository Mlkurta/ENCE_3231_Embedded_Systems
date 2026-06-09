
# Balancing Robot


## Introduction

This project is a two-wheel self-balancing robot built around an STM32 microcontroller and an MPU6050 inertial measurement unit. The robot uses real-time sensor fusion and closed-loop control to maintain an upright position while responding to user commands for forward, reverse, and turning motion.

The primary objective of the project is to explore the fundamentals of feedback control systems, embedded software development, and robotics. The balancing algorithm combines accelerometer and gyroscope measurements to estimate robot orientation, while a PID-based control loop continuously adjusts motor output to maintain stability.

This project serves as a platform for learning embedded systems, sensor integration, control theory, PCB design, and robotic system development. Future enhancements may include wheel encoders, improved state estimation, autonomous navigation, and advanced motion control techniques.

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

<img width="1140" height="1103" alt="Screenshot 2026-06-07 201333" src="https://github.com/user-attachments/assets/e614a03c-0016-47da-a7bb-72df90bb5e4f" />

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

It didnt. I ended up looking at the IC on the board and there was an interesting observation:

<img width="845" height="638" alt="IMG_8341" src="https://github.com/user-attachments/assets/4d985973-521e-4568-90d0-05c8bc0002c7" />


This was an AS5600. Upon looking at the datasheet, 0x32 is exactly the device address, but the read registers were different. In order to get raw angle, you must read 2 bytes beginning at 0x0C for angle. Then the degree conversion was different. Once that was determined, I had a means to measure angle and speed.


Balancing the Wheel velocities.


<img width="381" height="286" alt="image" src="https://github.com/user-attachments/assets/3240d63b-e6a9-4b09-9566-7023d75f9801" />


To characterize the drive motors and improve speed matching between the left and right wheels, a wheel speed calibration procedure was performed. The robot was elevated so the wheels could rotate freely, and PWM duty cycle values were applied in increments of 10 throughout the operating range. For each PWM setting, a video recording was taken while the wheel completed five full rotations. The recorded footage was then reviewed to determine the elapsed time required for the five rotations at each PWM value.

The measured rotation times were converted into wheel speed data and used to generate a relationship between PWM command and rotational speed. This calibration revealed variations in motor performance across the operating range and provided a basis for compensating differences between the two drive motors. By using the experimentally measured speed data, more consistent wheel velocities could be achieved, improving straight-line tracking and overall balancing performance of the robot.

