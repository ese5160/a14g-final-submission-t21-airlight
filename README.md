[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-24ddc0f5d75046c5622901739e7c5dd533143b0c8e959d652212380cedb1ea36.svg)](https://classroom.github.com/a/kzkUPShx)

# a14g-final-submission

    * Team Number: 21 
    * Team Name: AirLight
    * Team Members: Suraj Sree Balakrishna Marthy, Akhil Gunda
    * Github Repository URL: https://github.com/ese5160/a14g-final-submission-t21-airlight.git
    * Description of test hardware: AirLight PCBA and House Model, CCS811 Air Quality Sensor Development Board, LilyPad RGB LED, SG92R Servo Motor, J-Link, ASUS ROG STRIX G17(Windows Laptop), Macbook Air M2

## 1. Video Presentation

## 2. Project Summary

### Device Description

- Our device “AirLight” is an IoT (Internet of Things) based Air Quality Monitoring, Alerting and Actuating system incorporated with an Ambient light measurement and modulating system which could be utilized for continuous checking of the Air Quality (ppm/ppb) and Light Intensity(lux) levels at home.

- The IoT enabled device is coupled with sensors such as CCS811 (Air Quality IC)(offboard), VEML6030 (Ambient Light Sensor)(onboard) to detect; and actuators such as SG92R SERVO motor, Lilypad RGB LED to take necessary action based on the obtained data evaluated against set thresholds and standards.

- A Node Red dashboard has been implemented to display the monitored data and to communicate bi-directionally delivering Over-the-Air-Firmware-Updates(OTAFU) to the users.

### Inspiration

- Recent incident of two master’s students in the US - passed away due to Carbon Monoxide (CO) poisoning from air heater. A serious gas leak happens every 40 hours in the USA, claims more than 120 per year

- The smoke detectors that are generally installed in our houses do not detect the presence of any specific gas except for smoke and heat presence. This leads to false triggering or no alert at all in the case of invisible and non heat generating gas emissions which might be fatal. Very few people understand this and install gas alarms and gas detectors in their houses but that doesn't completely do the trick as well because it just alerts the user through the alarm being raised but no action is immediately taken to save the user. Many a times the user is not completely conscious during sleep even when the loud alarm is raised. So any delay in reacting to the gas presence by taking an evasive action could be critical and lead to death.

- So our IoT based smart solution tackles this problem by actuating a sliding door which improves the ventilation of the room and also turns on the fan acting like an exhaust. This mechanism gives the user a bit of leeway to escape or assess the situation and take corrective measures while the Air Quality Index of the room is being restored back to normal.

### Device Functionality

- AirLight has an Air Quality Index(AQI) Sensor mounted on one of the walls of the house and an onboard Ambient Light Sensor(ALS) on the PCBA. AQI sensor looks for the presence of Carbon Dioxide(CO2) and Total Volatile Organic Compounds(TVOC) in its vicinity and reports the values in parts per million (ppm) and parts per billion (ppb) respectively every 4.5 seconds to the SAMD21 microcontroller that is mounted and programmed on the PCBA. The ALS sensor gives the lux value every 5 seconds to the SAMD21.

- These readings are processed by the microcontroller in separate tasks out of which the AQI sensor readings have higher priority than the ALS sensor readings as the air quality index of the room is much more critical than the ambient light. When the CO2 level of the room goes above 3000 ppm or the TVOC level goes above 300 ppb (as per World Health Organization (WHO) recommended levels and permissible exposure limits (PEL)), the SERVO motor is actuated which is connected via a Rack & Pinion gear system to the sliding window/door of the room. The door is opened and the fan is turned on allowing for air flow to improve the air quality. In a case when the AQI readings are nominal and the ambient light levels are too low - <100 lux - the RGB LED is turned on facilitating higher illuminance levels in the room which is recommended as per international standards for living and bed rooms(100-200lux).

- The entire systems powered by a Li-Ion Battery that lasts for a continuous 5 hours since the nominal current consumption of the device is 300mA and the peak current consumption is 1.1A when the SERVO is actuated. This means that the battery rated to 3.7V with a full charged capacity of 2200mAh can power the device for around 7hrs, but considering the discharge characteristics of the Li-Ion battery, 5hrs of safe operation could be achieved. The IoT device has been tested for 3hrs of continuous operation without any issue.

- The obtained data of CO2 and TVOC values along with the lux levels in the room are continuously relayed to the MQTT based Node-Red Dashboard which displays the information to the user in real-time. History of the levels is stored on the Azure server and is timestamped and represented through the use of graphs. The user has an option of remotely opening the door of the house through the click of a button on the dashboard or could even turn on the RGB LED through an interactive button. The Node-Red Dashboard also features a functionality to push an over-the-air-firmware-upadte to the IoT Device with latest updates and features if any.

![DSD](assets/images/Detailed_System_Diag.jpeg)

### Challenges

- Most of our challenges were firmware and integration related:

  - CCS811 Air Quality Sensor Programming

    - The CCS811 Air Quality Sensor programming wasn't very straightforward and required a lot of registers on the module to be sensitively handled by writing correct sequences at the right instances to enable the sensing.

    - We had to go through a lot of datasheets and programming guides to get this to working. There was one guide that we found which was really helpful in understanding the flow of the sensor. But somehow the application binary that was present on the sensor was erased or was not responding to the register acceses.

    - We solved this through flashing the application binary onto the sensor using an Arduino ATMEGA board for which the code was available open-source on github. The flashed sensor started accessing the registers correctly and we were able to summon measurements from the sensor at specific time intervals.

  - VEML6030 Ambient Light Sensor Programming

    - The ambient light sensor requires repeated starts of I2C reads and writes which we didn't know as nothing of that sort was mentioned on the datasheet of this sensor.

    - Upon discussions with Detkin staff and TAs we came to know that a new block of code with repeated starts of I2CRead and I2CWrite was to be written for this purpose. Doing this solved the issue and we were able to take the sensor readings immmediately on the register specified.

  - Task Handling and Overflow

    - We were seeing a lot of stack overflows and program getting stuck after a actuating function was being called inside a sensor task.

    - We were able to solve this by playing with the stack sizes of different tasks and also by making sure that the tasks are not blocking each other. Sufficient delay between the tasks was essential in the smooth functionality which was achieved through vTaskDelay feature in FreeRTOS.

  - SERVO Motor Jitter and Functionality Errors

    - The SERVO we chose for opening and closing the window was not functioning properly as desired with complete rotation (in its possible range)

    - The reason was because the pulse widths modulation being passed on to it were not perfect and hence its behaviour was erratic. We fixed the issue with a very minute jitter still seen through incorporating for loops to repeatedly passing the PWM signal to the motor when required.

  - Laser Cutting / 3D Printing of the House Model

    - Initially we submitted 3D printing request for the House Model. Even though the model fit the size requirements of all the printers in Tangen Hall, Detkin Makerspace, Biotech Commons and Education Commons, the job time was exceeding 24 hours and hence our 3D printing request was getting denied.

    - A workaround we were able to figure out for this was laser cutting the walls, roof and the base of the house and joining them together using acrylic solvent. Another issue faced during this shift from 3D printing to Laser Cutting was that the dimensions created in the 3D CAD model differed from the acrylic sheets available for laser cutting. So a re-work on the CAD design was necessary. We were able to get the model together with all parts glued nicely together as desired.

### Prototype Learnings

- We were able to learn how a complete IoT product and solution to a prevalent issue could be devised and formulated in a short span of time while learning the intricacies of PCB design using Altium Designer and firmware programming using ATMEL Microchip Studio.

- Testing this device which is a custom PCBA helped us gain insights about how a microcontroller would behave when it doesn't have the circuitry that development board of the same microcontroller might have. We understood that accurate power circuitry and well routed signal (SPI) lines are crucial for this part as any changes in that would disengage the entire PCBA.

- Designing the CAD models using Solidworks was a great learning as we were unfamiliar with the software but knew the tools and features present in it from other softwares such as DS CATIA and OnShape.

- If we were to build this device again, I would try to incorporate alarm systems using buzzers, blinking hazard lamps and make a sophisticated air ventilation system like we all might have in our homes. Robust coding with real time SMS delivery via the Node-Red dashboard would be targeted to intimate emergency contacts and neighbors so that they could quickly attend to the issue.

### Next Steps for Improvement

- Inclusion of an SMS delivery mechanism would complete the loop in terms of alerting not just the user but their emergency contacts as well.

- Turning off the Air heating system which is causing the gas leak and integrating the control of the heating system with the IoT device.

- The data obtained from sensing the air quality and ambient light could be analyzed in terms of weather, environment and power consumption standards and used for improving device performance. Illuminance in the room could be improved and automated based on time-of-day along with the sensor measurements  

### Takeaways from ESE5160

- We learnt how a complete IoT solution could be designed and delivered in a short span of 3 months even with very less experience in PCB design and firmware programming.

- The assignments were crucial in enabling a progressive style of learning which helped us understand topics on the go while developing the project prototype at the same time.

- The lectures were important to understand the intricacies through experiences of the professor and the slides were very helpful to refer back to when some kind of design/debugging query arises.

### Project Links

 Node Red Instance Link : [Link to Node-Red Dashboard](http://52.254.80.220:1880/ui/#!/1?socketid=4tTUh_S3lkEitOhoAAAV)

 A12G code repository: [Link to A12G Github Repository](https://github.com/ese5160/a12g-firmware-drivers-t21-airlight.git)

 PCBA Altium 365 Link: [Link to Altium 365](https://upenn-eselabs.365.altium.com/designs/9070EB18-5774-43ED-B306-46CECA1679CC#design)

## 3. Hardware & Software Requirements

### 3.1 Hardware Requirements Specification (HRS)

The hardware requirements for the AirLight IoT-based solution product are outlined as follows:

#### HRS 01: Microcontroller Unit

- The project will be powered by the SAMW25 microcontroller, which integrates two co-processors: SAMD21G18A for processing and ATWINC1500 for WiFi capabilities.

#### HRS 02: Power Module

- The IoT device will employ the BQ24075 charging circuit, compatible with a 4.4V to 5.25V USB power source. This setup will be complemented by a 3.3V 2200mAh Lithium Ion battery. To ensure efficient operation, all components will be powered at 3.3V. A buck converter will be integrated into the system design to achieve stable 3.3V output.

#### HRS 03: Air Quality IC

- Air quality will be monitored using the CCS811 Air Quality IC (3.3V), measuring levels in ppm (parts per million) or ppb (parts per billion). The microcontroller will interact with this sensor via the I2C protocol. The sensor has a CO2 measurement range from 400ppm to 29206ppm and TVOC measurement from 0ppb to 32768ppb, with an accuracy of approximately ±5%.

#### HRS 04: Ambient Light Sensor

- Ambient light intensity will be monitored using the VEML6030 sensor (3.3V), communicating with the microcontroller via I2C. This onboard sensor has a dynamic range of 0 lux to ~140 kilo lux, offering high resolution down to 0.0042 lx/counts.

#### HRS 05: Servo Motor

- Based on readings from the CCS811 air quality sensor, a servo motor (3.3V) will be employed for actuation. The SAMW25 microcontroller will trigger the servo motor to open or close windows/doors to improve indoor air quality.

#### HRS 06: RGB LED Light

- Ambient lighting will be managed using an RGB LED light bulb (3.3V), controlled by the SAMW25 microcontroller based on data from the VEML6030 ambient light sensor.

These hardware components and their functionalities collectively enable the AirLight IoT solution to effectively monitor air quality and ambient conditions, facilitating automated actions to enhance indoor air quality and lighting levels.

### 3.2 Software Requirements Specification (SRS)

Here are the software requirements for the AirLight IoT-based solution product:

#### SRS 01: Interfacing with MCU, Sensors, and Actuators

- Microchip ATMEL Studio will serve as the primary platform for programming and connecting the SAMW25 microcontroller with the CCS811 air quality sensor, VEML6030 ambient light sensor, servo motor, and RGB LED bulb actuators.

#### SRS 02: Automatic Air Quality Reading

- The CCS811 Air Quality IC will be configured to detect gas presence at regular intervals (e.g., every second, ten seconds, or sixty seconds) in mode 1 for this project. The CO2 levels detected (ranging from 400ppm to 32768ppm) will be transmitted to the SAMW25 via I2C interface and processed using ATMEL Studio.

#### SRS 03: Automatic Light Intensity Reading

- The VEML6030 Ambient Light Sensor will detect ambient light levels within a dynamic range of 0 lux to ~140 kilo lux with high resolution. Light intensity data will be sent to the SAMW25 through I2C and handled using ATMEL Studio.

#### SRS 04: Servo Actuation Constraints

- The SAMW25 will trigger the servo motor when CO2 or TVOC levels exceed specified thresholds, facilitating window/door opening to improve air circulation. There will be checks in place to ensure the window/door is not already open, with servo angle adjusted accordingly.

#### SRS 05: RGB LED Actuation Constraints

- GPIO interface from the SAMW25 will control the RGB LED lights, toggling them based on ambient light levels falling below predefined thresholds.

#### SRS 06: Graphical User Interface (GUI Dashboard)

- The project will feature a user-friendly graphical interface to visualize air quality and light intensity. A Node-RED dashboard deployed on an Azure Virtual Machine will enable bi-directional communication, allowing users to monitor parameters, control actuators, and perform firmware updates remotely. This interface enhances user interaction and management of the IoT system.

## 4. Project Photos & Screenshots
