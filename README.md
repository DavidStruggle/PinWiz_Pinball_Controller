# PinWiz Pinball Controller
 ~ Purpuse Build Open Source Pinball Controller ~ 

## Features
- 10×10 Lamp driver Matrix ( LED and incandescent possible)
- 10×10 Switch matrix
- 40 High Power drivers
- 2.1ch Audio Amplifier  (2x 19W + 1x50W)
- 2ch General Illumination
- 2 Stepper Motor Drivers

## TODO
- Clean up schematics
- Include detailed BOM

## Background
I know that there are other “DIY” Pinball controllers like the PROC or the OPP, but I wanted to do my own (focused on fitting my pinball project). A big difference between my board and these boards is that i am using a central processor for doing everything rather than decentral boards, which do have their own processing unit on them. 

### Extract from my Blog
#### Audio Amplifier

The original game uses stereo music files and mono sound effects (sometimes panned to one speaker).  We also have a woofer in the cabinet, so it would make sense to have a tree way amplifier: two channels for the speakers, one for the woofer. Texas Instruments offers an IC, that fulfills our requirements. The TAS5755M is a very cool and relatively cheap chip. It is controlled by I2C and gets the sound data over standard I2S. I2S is very interesting, since we are basically sending a PCM signal to the codec. PCM is an uncompressed audio format found in wav files, which we are already have from the game files (more on that later). So we just need to read that file and send that exact data over to the codec without decompressing. This saves a lot of CPU load. Mixing multiple files is also easy since we can use simple addition (..and in some cases multiplication/division).

Another Bonus is, that the codec has an internal equalizer, which we can use to get the maximum sound quality out of our cabinet (..and adding a low pass filter to the subwoofer).

##### Stepper Motor Driver

Driving stepper motors is pretty straight forward: All you need are two H-Bridges and you are set (for an bipolar stepper motor). However, this will take some CPU load, handling the PWM signals to the bridges. While investigating, I found an interesting IC. The PCA9629A from NXP can control an uni polar stepper motor (with the help of some external components) over I2C. It has many features like ramping up and down, going x steps, reacting to interrupts and GPIO states. With the help of this IC, we can minimize another load on our processor. I also find it neat to see, that the datasheets states Amusement Machines as the first suggested use case 🙂

#### Solenoid drivers

There is not much to say for this one: The Solenoids will be fired by controlling an IRL540N mosfet in open drain configuration (it pulls the solenoid to ground). The mosfet is driven by a SN74ABT2827 buffer, which has multiple use cases: it is used as an watchdog, disabling all solenoids if the microprocessor has any problem, it is used to safe the microprocessor in case the mosfet fails and it is used as an level translator, since the microprocessor runs at 3.3v and the mosfet likes a gate-source voltage of around 5V.

#### Lamp Matrix drivers

Basically this is a standard TIP102 / TIP107 configuration as you see it on most pinball machines. The TIP102 NPN are buffered with a SN74ABT2827 and the TIP107 PNP are pulled low by ULN2003A darlington arrays. 

#### Switch Matrix drivers
The most simple solution yet: Just In/Out Buffers (SN74ABT2827) controlled by the microcontroller.

#### General Illumination
An IRL540N mosfet driven by a MCP1402 gate driver (used again as level translator). This mosfet will be PWM controlled. 

#### The Microcontroller
The heart of our machine. I have very pleasant experience with the STM32 series of microcontrollers so I stuck with them. The biggest factors here for choosing a microcontroller are:

    Amount of GPIOs
    I2S
    I2C
    SDIO (..at least SPI)
    SPI 

There are quite a few microcontrollers which fulfill our criteria. Luckily, most of them pin compatible. Candidates are : STM32F469 (IE, II or IG) STM32F767 (II or IG), or even STM32H743 (II or IG).

I chose the STM32F767 – it should be more than enough powerful for what we will use it. You could however choose another controller if you like. Porting the source code should not be a problem, since they are all backwards compatible.

#### Miscellaneous

An SD card is directly connected to the microcontroller. The DMD will also be driven directly from the microcontroller. A backup SRAM battery has been added to store variables in SRAM and keep time and date. I probably wont end up using the SRAM for high scores, statistics and options, since its volitale. These things will probably end up in Flash or on the SD (which should be fine, since they are not written all the time). The microcontroller will be programmable over JTAG or SWD. In this case, I have gone with SWD, since it works great on the Nucleo boards. 
