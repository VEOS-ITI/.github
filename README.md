# VEOS: Vehicle Embedded Operating System

## Introduction

VEOS (Vehicle Embedded Operating System) is an innovative project developed by the Embedded Systems Track at the Information Technology Institute (ITI). It aims to redefine the driving experience by integrating advanced car control, enhanced safety, and robust security features within a multi-display in-car system. This system provides real-time vehicle data visualization, smart assistance capabilities, and remote access through a dedicated mobile application, ultimately improving safety, convenience, and user engagement both inside and outside the vehicle.

Modern vehicles are evolving into sophisticated, connected platforms. However, current in-car systems often suffer from complexity, limited feature flexibility, and a lack of remote accessibility. VEOS addresses these challenges by offering a comprehensive solution that transforms the traditional driving experience into a more intuitive, interactive, and secure journey. The project's motivation stems from the desire to provide a luxurious and feature-rich car environment that aligns with contemporary technological advancements, ensuring a superior user experience and a secure interaction with the vehicle.

## Features

VEOS is equipped with a suite of features designed to enhance every aspect of vehicle interaction:

### Multi-display System

VEOS incorporates a sophisticated multi-display setup, featuring a dedicated dashboard for critical real-time vehicle statistics and a separate driver screen for infotainment and other applications. This dual-screen approach ensures that essential driving information is always accessible, while also providing a rich, customizable environment for entertainment and auxiliary functions.

### Remote Access with VEOS Mobile App

The VEOS Mobile App offers comprehensive remote access capabilities, allowing users to monitor vehicle status, control vehicle limits, and track performance from anywhere. This empowers drivers with complete visibility and control, enabling smarter driving decisions, improved safety, and an enhanced overall driving experience. The app features dynamic UI updates, reflecting real-time vehicle status, and allows for remote restrictions such as speed limit control and interior temperature management.

### Enhanced Safety and Security

Security is a cornerstone of the VEOS design. The system integrates several robust mechanisms to ensure a secure and reliable driving experience:

*   **RSA-248 Encryption**: Provides secure authentication and prevents tampering with transmitted data across the network, particularly when receiving values from the Logitech system on the AOSP side.
*   **Checksum Validation**: Every command packet includes a checksum, which is verified upon reception. Packets with mismatched checksums are discarded, preventing corrupted or malicious data from affecting the vehicle's operation.
*   **Fixed IP Addressing**: Each subsystem is assigned a static IP address, ensuring that only known and authorized devices can connect to the network.
*   **Isolated Network Design**: Only whitelisted devices are permitted on the dedicated network, significantly reducing the risk of external intrusions.

### Seamless Connectivity

VEOS ensures continuous connectivity through the integration of an LTE Modem, providing reliable internet access for the in-car system. This connectivity supports a wide range of telematics and smart vehicle features, including enhanced infotainment, navigation, Over-the-Air (OTA) updates, and robust Car-to-Cloud (C2C) communication for monitoring and data storage.

## System Architecture

VEOS is built upon a modular architecture comprising three main nodes, each playing a crucial role in the system's overall functionality:

### 1. Car Side (Bare-metal)

This node is based on an STM32 Microcontroller and is responsible for the fundamental control of the vehicle's movement and basic functions. It interprets inputs from the Logitech system and translates them into actions such as motion control, steering, and activation of auxiliary features like lights and horn. The Car Control subsystem ensures that the physical vehicle responds reliably and safely to both manual and remote commands.

**Key Features:**

*   **Motion Control**: Allows the car to move forward and backward based on user input.
*   **Steering Control**: Enables angle adjustments for turning the car.
*   **Lighting & Horn Control**: Provides direct control over the car’s headlights, indicators, and horn.
*   **Input Handling**: Receives commands from the Logitech steering wheel to manage all vehicle actions.

**Technology Stack (Hardware & Firmware):**

*   **Hardware**: STM32 Microcontroller, Ultrasonic Sensors (for obstacle detection), Motor Driver & Encoder (for motor operation and wheel movement tracking), UART-to-USB Interface.
*   **Firmware**: STM32 HAL (Hardware Abstraction Layer) for low-level hardware control, C Language for firmware development, and a custom Packet Protocol for reliable communication.

### 2. Yocto Image #1 & #2

These Yocto-based images serve as middleware platforms, bridging communication between the vehicle hardware, the Android system (AOSP), and external services. They are central to handling commands, managing data flow, and ensuring smooth integration of different subsystems.

**Yocto Image #1 (Logitech to STM32 & Camera Integration):**

This image receives control commands from the Logitech system using the SOME/IP protocol, processes them, and forwards relevant instructions to the STM32 microcontroller via UART. It also supports camera integration by capturing, compressing (H.264), and streaming video via an RTSP server, allowing the AOSP side to access live video.

**Key Features:**

*   **Command Handling**: Receives and processes commands from the Logitech system.
*   **STM32 Communication**: Uses UART to send processed commands to the microcontroller for real-time control.
*   **Camera Integration**: Captures camera feeds, compresses video, and hosts an RTSP server for live video streaming to Android applications.
*   **Networking Support**: Utilizes network protocols for timing synchronization and minimizing latency.

**Solved Issues:**

*   **Latency Reduction**: Implemented timing synchronization protocols for smooth, real-time video streaming.
*   **Reliable Communication**: Established stable data transfer between Logitech, Yocto, and STM32 via UART.
*   **Switch Optimization**: Connected two network switches in a peer-to-peer configuration to reduce bottlenecks.

**Yocto Image #2 (Logitech to AOSP):**

This image acts as a middleware between the Logitech system and the AOSP environment. It receives data and control signals from Logitech, processes them, and transmits the results to AOSP via the SOME/IP protocol. This enables real-time updates on the dashboard screen, visualizing speed, gear status, and other parameters.

**Key Features:**

*   **Logitech Integration**: Interfaces directly with the Logitech system for vehicle data and driver commands.
*   **AOSP Communication**: Uses SOME/IP to transmit telemetry and user inputs to AOSP for dashboard visualization.
*   **Data Synchronization**: Ensures consistent updates between Logitech inputs and AOSP display.

### 3. AAOS (Android Automotive OS)

The Android Automotive OS (AAOS) node is where the dashboard and infotainment screens are developed, providing a user-friendly interface for the driver. It interacts with the Vehicle HAL (VHAL) to access key vehicle properties and communicates with the Logitech system via SOME/IP for continuous updates.

**Dashboard Screen:**

The dashboard screen displays real-time vehicle data, including speed, gear, battery level, car lighting, and tire pressure. It dynamically updates data from VHAL and SOME/IP communication and also retrieves speed limit values from a cloud server, enhancing safety and control.

**Key Features:**

*   **Real-time Updates**: Displays car status (speed, gear, battery, lighting, tire pressure).
*   **Dynamic Monitoring**: Continuous refresh of data from VHAL and SOME/IP.
*   **Speed Limit Display**: Shows speed limits retrieved from the server.
*   **Warnings and Alerts**: Visualizes warnings for vehicle problems.

**Technology Stack:**

*   **Platform**: AOSP (Android Open Source Project).
*   **Interfaces**: Vehicle HAL (VHAL), Shared Library (bridge between VHAL and UI).
*   **Communication**: SOME/IP Protocol, Common API.
*   **Cloud Integration**: Cloud Server for speed limits and telematics data.

**Infotainment Screen:**

The infotainment screen includes applications like the Camera App and VEOS Radio App.

**Camera App:**

Allows the user to view front and rear camera feeds. The camera view dynamically switches based on the gear status (front camera for 'D', rear camera for 'R').

**Key Features:**

*   **Low-Latency RTSP Streaming**: Streams live video feeds from IP cameras using LibVLC.
*   **Multiple Camera Support**: Utilizes hardware decoding for smoother playback.
*   **Error Handling & Auto-Restart**: Automatically restarts playback if the stream ends.

**VEOS Radio App:**

Integrates FM radio functionality, providing seamless access to live radio stations. It uses a custom hardware module (TEA5767 FM tuner chip) and is tightly coupled with the Android framework.

**Key Features:**

*   **Custom FM Radio Module**: Based on TEA5767 FM tuner chip.
*   **Radio Controls**: Tune stations, mute/unmute, power on/off via app.
*   **Seamless Integration**: Designed for Android 15 kernel, communicates via dedicated service and HAL layer.

## Technology Stack

*   **Hardware**: STM32 Microcontroller, Raspberry Pi 3, Raspberry Pi 5, Quectel EC200U-EU 4G LTE modem.
*   **Software**: Yocto, Android Automotive OS (AAOS), FreeRTOS.
*   **Communication Protocols**: SOME/IP, UART, RTSP.
*   **Mobile App**: Android, Kotlin, Retrofit.

## Getting Started

To get a local copy up and running, follow these simple steps.

### Prerequisites

*   [List any software or hardware prerequisites here]

### Installation

1.  Clone the repository:
    ```bash
    git clone https://github.com/your-username/VEOS.git
    ```
2.  [Add detailed installation steps for each component: Car Side, Yocto Images, AAOS, Mobile App]

## Usage

[Provide instructions on how to use the VEOS system, including how to interact with the multi-display system, mobile app, and other features.]

## Contributing

Contributions are what make the open-source community such an amazing place to learn, inspire, and create. Any contributions you make are **greatly appreciated**.

1.  Fork the Project
2.  Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3.  Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4.  Push to the Branch (`git push origin feature/AmazingFeature`)
5.  Open a Pull Request

## License

Distributed under the MIT License. See `LICENSE` for more information.

## Contact

*   Ahmed Mohamed Adly Sultan
*   Ahmed Yousry Ibrahim Al Morady
*   Aliaa Ahmed Mortada Abd Alfatah
*   Yara Abdelraheem Omran Abdelraheem

Project Link: [https://github.com/your-username/VEOS](https://github.com/your-username/VEOS)

## Acknowledgments

*   Dr. Youssef Nofal (Supervisor)
*   Our teaching assistants
*   Our families and friends




### System Architecture Diagrams

Below are the detailed system architecture diagrams illustrating the interconnectedness of VEOS components.

![System Architecture Diagram 1](https://github.com/user-attachments/assets/25155c4c-e9ea-4e0b-a4b6-9600e87169b9)




### Multi-display System Visuals

The multi-display system is a core component of VEOS, providing distinct interfaces for the driver. The dashboard offers a clear view of real-time car statistics, including speed, battery level, gear status, turn signals, and lighting controls, ensuring the driver always stays informed and in control. The driver screen is isolated from the dashboard and runs separate applications dedicated to the driver, such as the VEOS Radio app and other installed applications.

![Dashboard Screen](https://private-us-east-1.manuscdn.com/sessionFile/B8B5ILTkfCmGIS6LrtB5Q4/sandbox/W5Q3RrpxXo2kZKyyTLMeiY-images_1758388030251_na1fn_L2hvbWUvdWJ1bnR1L1JFQURNRV9pbWFnZXMvZGFzaGJvYXJkX3NjcmVlbg.webp?Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9wcml2YXRlLXVzLWVhc3QtMS5tYW51c2Nkbi5jb20vc2Vzc2lvbkZpbGUvQjhCNUlMVGtmQ21HSVM2THJ0QjVRNC9zYW5kYm94L1c1UTNScnB4WG8ya1pLeXlUTE1laVktaW1hZ2VzXzE3NTgzODgwMzAyNTFfbmExZm5fTDJodmJXVXZkV0oxYm5SMUwxSkZRVVJOUlY5cGJXRm5aWE12WkdGemFHSnZZWEprWDNOamNtVmxiZy53ZWJwIiwiQ29uZGl0aW9uIjp7IkRhdGVMZXNzVGhhbiI6eyJBV1M6RXBvY2hUaW1lIjoxNzk4NzYxNjAwfX19XX0_&Key-Pair-Id=K2HSFNDJXOU9YS&Signature=r67fC7dP1Bwf7hD5Z0U2LSyLZbX5~pC2mOqBMdNCWH8NUw25JW3i9Pz4gN76FZFZ7J88hgOn1XKDkNcLozCXinOoCpVB3upz7mAbAtDbapdTiq38yUTTwwe3qFYDeWL4RMKcUXZFc5HQuA61ZX8Y6BAP2hU1Ybzcqx--0Q57A2ZxlgGPctFBQMaAVW-XNwKWlXlf1xMbl5UBpncGo10h745eHztPpZNGLfkkdy3vynnW6~q96cRLGYi-CUkqOot6vYjTzADqlbDYs9vtHiOuTC24DflbDLyjShnmsE7SXQB5ggP-15LZ-8kKc3lp4uQ6h1VNMvyPynERJyo9mu6Esw__)
*Figure: VEOS Dashboard Screen displaying real-time vehicle data.*

![Driver Screen](https://private-us-east-1.manuscdn.com/sessionFile/B8B5ILTkfCmGIS6LrtB5Q4/sandbox/W5Q3RrpxXo2kZKyyTLMeiY-images_1758388030252_na1fn_L2hvbWUvdWJ1bnR1L1JFQURNRV9pbWFnZXMvZHJpdmVyX3NjcmVlbg.webp?Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9wcml2YXRlLXVzLWVhc3QtMS5tYW51c2Nkbi5jb20vc2Vzc2lvbkZpbGUvQjhCNUlMVGtmQ21HSVM2THJ0QjVRNC9zYW5kYm94L1c1UTNScnB4WG8ya1pLeXlUTE1laVktaW1hZ2VzXzE3NTgzODgwMzAyNTJfbmExZm5fTDJodmJXVXZkV0oxYm5SMUwxSkZRVVJOUlY5cGJXRm5aWE12WkhKcGRtVnlYM05qY21WbGJnLndlYnAiLCJDb25kaXRpb24iOnsiRGF0ZUxlc3NUaGFuIjp7IkFXUzpFcG9jaFRpbWUiOjE3OTg3NjE2MDB9fX1dfQ__&Key-Pair-Id=K2HSFNDJXOU9YS&Signature=jUG2yHWVDICnwUS~GOTZaew71g-nxiSEDdsEwPE2hmqr57CgvggMmYIKbtdFN~MfEFomr-1nKBEZN2bdtVfIFUeef1iVjcbb3-hPYt9-P2ZdfVp3sNr0F7NE8WWRRrVtf36753jhg~EjsCbWpj1DRzWNUMOTiG-TTzx9plorh7bD5-4pebp0~MYPiZwKS3aZrbxBSNqDEktQrDfUmNvm28LYSqE2zK2p3qr8zmilkUDl0f6522Smu9cU~q-7xe5YIcaNvKWDHt58zSHzwU1LAHP8hZ89lRckgGsm8ytxfJoAL36XRxl1ZrIToWDHIvlBkzDUQfh1TBX8UY~Zi2w7Mw__)
*Figure: VEOS Driver Screen showcasing infotainment features.*

### Remote Access with VEOS Mobile App Visuals

The VEOS Mobile App allows users to monitor various aspects of their vehicle remotely. This includes real-time vehicle status (driving, parking, charging), comprehensive vehicle metrics (battery level, trip duration, current speed, mileage, battery temperature, remaining driving range, maximum speed reached), and dynamic UI updates that change according to the vehicle's status. Users can also set custom restrictions like speed limits and interior temperature limits.

![VEOS Mobile App Screenshot](https://private-us-east-1.manuscdn.com/sessionFile/B8B5ILTkfCmGIS6LrtB5Q4/sandbox/W5Q3RrpxXo2kZKyyTLMeiY-images_1758388030254_na1fn_L2hvbWUvdWJ1bnR1L1JFQURNRV9pbWFnZXMvbW9iaWxlX2FwcF9zY3JlZW5zaG90.webp?Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9wcml2YXRlLXVzLWVhc3QtMS5tYW51c2Nkbi5jb20vc2Vzc2lvbkZpbGUvQjhCNUlMVGtmQ21HSVM2THJ0QjVRNC9zYW5kYm94L1c1UTNScnB4WG8ya1pLeXlUTE1laVktaW1hZ2VzXzE3NTgzODgwMzAyNTRfbmExZm5fTDJodmJXVXZkV0oxYm5SMUwxSkZRVVJOUlY5cGJXRm5aWE12Ylc5aWFXeGxYMkZ3Y0Y5elkzSmxaVzV6YUc5MC53ZWJwIiwiQ29uZGl0aW9uIjp7IkRhdGVMZXNzVGhhbiI6eyJBV1M6RXBvY2hUaW1lIjoxNzk4NzYxNjAwfX19XX0_&Key-Pair-Id=K2HSFNDJXOU9YS&Signature=ClfVRe3NfxnLBKUoHjdTJnAvU3b-GTF04nnhaoPJQoUwmN9ijSb2AUoHMoXmmPHX1dszRrs9WmvTJlV8YCmvctUr-8BrokcsTGRf9SpS3SIdx~u0SPrJvD~feyK0NSzYg984hfCtM4VEfJySQtTi4mw6GT97nnb~qLAGCxvkoqg0CSY0mMNdvefpk19g2zn8u9PSCe5Niv7ARU39ou5VexSmI4GhFajaM6wr5~lsw-o5V1wOGn4-32wFaFu8uscvQ8-YLEIH56pidIBmA77HBWiPVpV3oTcxUuOXriam6coRfPZRzy-CRir4xVw6eJ5PTw~ggXHn2971V5-DkkTsCQ__)
*Figure: VEOS Mobile App interface for vehicle monitoring and control.*

### In-Car Applications

VEOS integrates several in-car applications to enhance the user experience.

#### VEOS Radio App

The VEOS Radio App provides seamless access to live FM radio stations, controlled directly from the infotainment screen. It is built with a custom hardware module based on the TEA5767 FM tuner chip and integrates tightly with the Android framework.

![VEOS Radio App](https://private-us-east-1.manuscdn.com/sessionFile/B8B5ILTkfCmGIS6LrtB5Q4/sandbox/W5Q3RrpxXo2kZKyyTLMeiY-images_1758388030318_na1fn_L2hvbWUvdWJ1bnR1L1JFQURNRV9pbWFnZXMvcmFkaW9fYXBw.webp?Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9wcml2YXRlLXVzLWVhc3QtMS5tYW51c2Nkbi5jb20vc2Vzc2lvbkZpbGUvQjhCNUlMVGtmQ21HSVM2THJ0QjVRNC9zYW5kYm94L1c1UTNScnB4WG8ya1pLeXlUTE1laVktaW1hZ2VzXzE3NTgzODgwMzAzMThfbmExZm5fTDJodmJXVXZkV0oxYm5SMUwxSkZRVVJOUlY5cGJXRm5aWE12Y21Ga2FXOWZZWEJ3LndlYnAiLCJDb25kaXRpb24iOnsiRGF0ZUxlc3NUaGFuIjp7IkFXUzpFcG9jaFRpbWUiOjE3OTg3NjE2MDB9fX1dfQ__&Key-Pair-Id=K2HSFNDJXOU9YS&Signature=dqJ2BhBW29jkxn6Jt4X5qKUeiqlkda~1vJZecJA21hnRbMzEMGZlfjXJtxHRoCsvhum-ySuRTav8WGCdnJUOBPD63AGWqdb5DujOfvfJUukJayWb~9t-uOMXgofzXEFxr1odk63SL9Wwz~SP5qXUJBN-Sm8TvAZ8WK0KjdlDuNYXrQVQc7yeBapRd3cdRO8PF1nMgoycwY0CmwT1wJIhfYaqajxRzTXCT53qPjsTvxaU068C3W8~g6jdyreshLL865tJDav-tVbx86dsAz2-rn7TPQlLV9DDdUKZvFml870VjA9lTrulT2jBW1xC4yF1WHyyvirtG~iesdbhD-jrmA__)
*Figure: VEOS Radio App interface on the infotainment screen.*

#### Camera App

The Camera App allows drivers to view live feeds from front and rear cameras, dynamically switching based on the vehicle's gear status. This feature enhances safety by providing clear visibility around the car.

![VEOS Camera App](https://private-us-east-1.manuscdn.com/sessionFile/B8B5ILTkfCmGIS6LrtB5Q4/sandbox/W5Q3RrpxXo2kZKyyTLMeiY-images_1758388030319_na1fn_L2hvbWUvdWJ1bnR1L1JFQURNRV9pbWFnZXMvY2FtZXJhX2FwcA.webp?Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9wcml2YXRlLXVzLWVhc3QtMS5tYW51c2Nkbi5jb20vc2Vzc2lvbkZpbGUvQjhCNUlMVGtmQ21HSVM2THJ0QjVRNC9zYW5kYm94L1c1UTNScnB4WG8ya1pLeXlUTE1laVktaW1hZ2VzXzE3NTgzODgwMzAzMTlfbmExZm5fTDJodmJXVXZkV0oxYm5SMUwxSkZRVVJOUlY5cGJXRm5aWE12WTJGdFpYSmhYMkZ3Y0Eud2VicCIsIkNvbmRpdGlvbiI6eyJEYXRlTGVzc1RoYW4iOnsiQVdTOkVwb2NoVGltZSI6MTc5ODc2MTYwMH19fV19&Key-Pair-Id=K2HSFNDJXOU9YS&Signature=EFmojld7t2uYJ1rNtt99JZS9kKSW7V3TaQk4DQmADqSTrtxWKsO051RsuGPTl7SMoVJM3~kqlQtCkSLBsOioEqcFUdQKiRZhcWBHJM79-TjFH63BUZ9XvzzEl6CHQMY9fc-RDhbeYHalziDr~Gq3xxa~lNqW-NY1JF~~AUi1U9k4h28WSTR7y1Ca6Dqlo5EIgeLco6JLUFy6yfDVHQ8VtV5KnoisP7DiDR83uKQg6kstc0PxqDWDjIJouFBbxagSR7CHHnBiKWU5IUsPk3ovIuKEq9uURjz9TJVLvVHy29dJCCQ83Y1nULVVFk0BOpxWUswi0zgXyBWhUWI2OI5ZSg__)
*Figure: VEOS Camera App displaying a live feed.*

### Car Side (Bare-metal) Visuals

The bare-metal car control system is responsible for executing driver commands and managing vehicle movement. It interfaces with input devices like the Logitech steering wheel.

![Logitech Control](https://private-us-east-1.manuscdn.com/sessionFile/B8B5ILTkfCmGIS6LrtB5Q4/sandbox/W5Q3RrpxXo2kZKyyTLMeiY-images_1758388030321_na1fn_L2hvbWUvdWJ1bnR1L1JFQURNRV9pbWFnZXMvbG9naXRlY2hfY29udHJvbA.webp?Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9wcml2YXRlLXVzLWVhc3QtMS5tYW51c2Nkbi5jb20vc2Vzc2lvbkZpbGUvQjhCNUlMVGtmQ21HSVM2THJ0QjVRNC9zYW5kYm94L1c1UTNScnB4WG8ya1pLeXlUTE1laVktaW1hZ2VzXzE3NTgzODgwMzAzMjFfbmExZm5fTDJodmJXVXZkV0oxYm5SMUwxSkZRVVJOUlY5cGJXRm5aWE12Ykc5bmFYUmxZMmhmWTI5dWRISnZiQS53ZWJwIiwiQ29uZGl0aW9uIjp7IkRhdGVMZXNzVGhhbiI6eyJBV1M6RXBvY2hUaW1lIjoxNzk4NzYxNjAwfX19XX0_&Key-Pair-Id=K2HSFNDJXOU9YS&Signature=Px4Rb5gbg~QAdnpYOrN5Kg1KumJ6bSrQp2~KBdYvj8P08gosEN-RifRr5QGydgfAeT76q8dO0LoFfiBCndtKO6InfMYzPNjferBwiH3yH3STC~XKEnuXtgXkyLx06v3ZerhRjx4sEe-lT2OEwlQEKnVeNr7868~btFQfSJ1uLmbIJo7WhhwwUrNRV7kIRTuJvUk49aSELbGqYQWnJEay5hWxWhneAu5f7rghk1JhEDJMTXHnxFC-py7GKO63yXI9MVqJmMptNiW9GMKVobP3mjRuVHiwStz5EamMyLCrgRaX30iYvsVtMzZTyYuTfkDF0frpaOqigzM~6Thc~l8KkQ__)
*Figure: Logitech steering wheel setup for vehicle control.*

![Car Bare Metal](https://private-us-east-1.manuscdn.com/sessionFile/B8B5ILTkfCmGIS6LrtB5Q4/sandbox/W5Q3RrpxXo2kZKyyTLMeiY-images_1758388030322_na1fn_L2hvbWUvdWJ1bnR1L1JFQURNRV9pbWFnZXMvY2FyX2JhcmVfbWV0YWw.webp?Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9wcml2YXRlLXVzLWVhc3QtMS5tYW51c2Nkbi5jb20vc2Vzc2lvbkZpbGUvQjhCNUlMVGtmQ21HSVM2THJ0QjVRNC9zYW5kYm94L1c1UTNScnB4WG8ya1pLeXlUTE1laVktaW1hZ2VzXzE3NTgzODgwMzAzMjJfbmExZm5fTDJodmJXVXZkV0oxYm5SMUwxSkZRVVJOUlY5cGJXRm5aWE12WTJGeVgySmhjbVZmYldWMFlXdy53ZWJwIiwiQ29uZGl0aW9uIjp7IkRhdGVMZXNzVGhhbiI6eyJBV1M6RXBvY2hUaW1lIjoxNzk4NzYxNjAwfX19XX0_&Key-Pair-Id=K2HSFNDJXOU9YS&Signature=WAOm3hB9yMmO0ADSTo6RBXBHZvtahfXybBdLMxOvrb0Pck9SAkHRr99JZw8FppTQCJoJeeM3Ri3iupfHGiVHH3z5dTSKmYOmpm3ehoZ3hEf4dI37P93Jw1NqNb9OKuCk8jHhG-1OYpzqyxDsKa2j3-X2p7zpgi5zgpBnrmwu5lNQjGur291BiBuyixG7FUiHjCDgYqQTsx3lGjap5qqkUrGY~YukD46SnxSfD5yUW~BrKo8TAS5fCBpP3DprKrR7CZ8RkicXLl14TcDf1N-cXE2nmWIepPmYh8YP6~6dPhMBX6sMA9mJqzlDW0S5b4hGIF49kOaw3-3700FrQ-WKBw__)
*Figure: Physical setup of the car's bare-metal control system.*


