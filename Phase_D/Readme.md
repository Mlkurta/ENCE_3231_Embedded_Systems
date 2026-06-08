
# Balancing Robot


## Inroduction

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

## PCB Design

<img width="1205" height="804" alt="image" src="https://github.com/user-attachments/assets/9ea69e72-081f-49be-bcbd-0ee2c5b34d3d" />

One error I made is I did not use a separate name for the 5V voltage regulator; I used the same LDR for 3.3V and 5V. Fortunately the footprins are the same, and the correct parts were available when soldering.


## Encoder Problems

<img width="590" height="1278" alt="IMG_8338" src="https://github.com/user-attachments/assets/4e348e50-2e90-4213-b8ac-f64708f39fdc" />


Device ID

<img width="590" height="1278" alt="IMG_8339" src="https://github.com/user-attachments/assets/c4efb273-a7b9-4eac-8fbb-25913529fc98" />

This showed a device ID (decimal = 54, which is Hex 0x36)

HAL == OK

<img width="590" height="1278" alt="IMG_8340" src="https://github.com/user-attachments/assets/3e776125-f507-4518-93e8-705dff1c63ef" />

