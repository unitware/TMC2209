# TMC2209 Library

**Original Repository:** https://github.com/janelia-arduino/TMC2209  
**Original Author:** Peter Polidoro <peter@polidoro.io>  
**Original License:** BSD  

This is a fork of the original TMC2209 library, modified for Pico SDK compatibility.  
All original copyright notices and BSD license terms are preserved.



## Description

The TMC2209 is an ultra-silent motor driver IC for two phase stepper motors with both UART serial and step and direction interfaces.


## Communication

The TMC2209 driver has two interfaces to communicate with a stepper motor controller, a UART serial interface and a step and direction interface.

The UART serial interface may be used for tuning and control options, for diagnostics, and for simple velocity commands.

The step and direction interface may be used for real-time position, velocity, and acceleration commands.

### Unidirectional Communication

TMC2209 parameters may be set using unidirectional communication from a microcontroller UART serial TX pin to the TMC2209 PDN\_UART pin. Responses from the TMC2209 to the microcontroller are ignored.

<img src="./images/trinamic_wiring-TMC2209-unidirectional.svg" width="1920px">


### Bidirectional Communication

The UART single wire interface allows control of the TMC2209 with any set of microcontroller UART serial TX and RX pins.

1.  Coupled

    The simpliest way to connect the single TMC2209 serial signal to both the microcontroller TX pin and RX pin is to use a 1k resistor between the TX pin and the RX pin to separate them.
    
    Coupling the TX and RX lines together has the disadvantage of echoing all of the TX commands from the microcontroller to the TMC2209 on the microcontroller RX line. These echos need to be removed by this library in order to properly read responses from the TMC2209.
    
    Another disadvantage to coupling the TX and RX lines together is that it limits the length of wire between the microcontroller and the TMC2209. The TMC2209 performs a CRC (cyclic redundancy check) which helps increase interface distances while decreasing the risk of wrong or missed commands even in the event of electromagnetic disturbances.
    
    <img src="./images/trinamic_wiring-TMC2209-bidirectional-coupled.svg" width="1920px">

### Serial Baud Rate

No baud rate configuration on the chip is required, as the TMC2209 automatically adapts to the baud rate. 9600 to 500000 Baud have been tested and work. Default is 115200.


## Step and Direction Interface



# Settings

The default settings for this library are not the same as the default settings for the TMC2209 chip during power up.

The default settings for this library were chosen to be as conservative as possible so that motors can be attached to the chip without worry that they will accidentally overheat from too much current before library settings can be changed.

These default settings may cause this library to not work properly with a particular motor until the settings are changed.

The driver starts off with the outputs disabled, with the motor current minimized, with analog current scaling disabled, and both the automatic current scaling and automatic gradient adaptation disabled, and the cool step feature disabled.

Change driver settings with care as they may cause the motors, wires, or driver to overheat and be damaged when the current is too high. The driver tends to protect itself and shutdown when it overheats, then reenable when the driver cools, which can result in odd jerky motor motion.


## Driver Enable

The driver is disabled by default and must be enabled before use.

The driver may be disabled in two ways, either in hardware or in software, and the driver must be enabled in both ways in order to drive a motor.

To enable the driver in software, or optionally in both hardware and software, use the enable() method.

To disable the driver in software, or optionally in both hardware and software, use the disable() method.


### Hardware Enable

The TMC2209 chip has an enable input pin that switches off the power stage, all motor outputs floating, when the pin is driven to a high level, independent of software settings.

The chip itself is hardware enabled by default, but many stepper driver boards pull the enable input pin high, which causes the driver to be disabled by default.

To hardware enable the driver using this library, use the setHardwareEnablePin method to assign a microcontroller pin to contol the TMC2209 enable line.

To hardware enable the driver without using this library, pull the enable pin low, either with a jumper or with an output pin from the microcontroller.

The method hardwareDisabled() can be used to tell if the driver is disabled in hardware.


### Software Enable

The TMC2209 may also be enabled and disabled in software, independent of the hardware enable pin.

When the driver is disabled in software it behaves the same as being disabled by the hardware enable pin, the power stages are switched off and all motor outputs are floating.

This library disables the driver in software by default.


## Analog Current Scaling

Analog current scaling is disabled in this library by default, so a potentiometer connected to VREF will not set the current limit of the driver. Current settings are controlled by UART commands instead.

Use enableAnalogCurrentScaling() to allow VREF, the analog input of the driver, to be used for current control.

According to the datasheet, modifying VREF or the supply voltage invalidates the result of the automatic tuning process. So take care when attempting to use analog current scaling with automatic tuning at the same time.


## Automatic Tuning

The TMC2209 can operate in one of two modes, the voltage control mode and the current control mode.

In both modes, the driver uses PWM (pulse width modulation) to set the voltage on the motor coils, which then determines how much current flows through the coils. In voltage control mode, the driver sets the PWM based only on the driver settings and the velocity of the motor. In current control mode, the driver uses driver settings and the velocity to set the PWM as well, but in addition it also measures the current through the coils and adjusts the PWM automatically in order to maintain the proper current levels in the coils.

Voltage control mode is the default of this library.

The datasheet recommends using current control mode unless the motor type, the supply voltage, and the motor load, the operating conditions, are well known. This library uses voltage control mode by default, though, because there seem to be cases when the driver is unable to calibrate the motor properties properly and that can cause the motor to overheat before the settings are adjusted.

The datasheet explains how to make sure the driver performs the proper automatic tuning routine in order to use current control mode.

Use enableAutomaticCurrentScaling() to switch to current control mode instead.


### Voltage Control Mode

Use disableAutomaticCurrentScaling() to switch to voltage control mode and disable automatic tuning.

When automatic current scaling is disabled, the driver operates in a feed forward velocity-controlled mode and will not react to a change of the supply voltage or to events like a motor stall, but it provides a very stable amplitude.

When automatic tuning is disabled, the run current and hold current settings are not enforced by regulation but scale the PWM amplitude only. When automatic tuning is disabled, the PWM offset and PWM gradient values may need to be set manually in order to adjust the motor current to proper levels.


### Current Control Mode

Use enableAutomaticCurrentScaling() to switch to current control mode and enable automatic tuning.

Use enableAutomaticGradientAdaptation() when in current control mode to allow the driver to automatically adjust the pwm gradient value.

When the driver is in current control mode it measures the current and uses that feedback to automatically adjust the voltage when the velocity, voltage supply, or load on the motor changes. In order to respond properly to the current feedback, the driver must perform a calibration routine, an automatic tuning procedure, to measure the motor properties. This allows high motor dynamics and supports powering down the motor to very low currents.

Refer to the datasheet to see how to make the driver perform the automatic tuning procedure properly.


### PWM Offset

The PWM offset relates the motor current to the motor voltage when the motor is at standstill.

Use setPwmOffset(pwm\_amplitude) to change. pwm\_amplitude range: 0-255

In voltage control mode, increase the PWM offset to increase the motor current.

In current control mode, the pwm offset value is used for initialization only. The driver will calculate the pwm offset value automatically.


### PWM Gradient

The PWM gradient adjusts the relationship between the motor current to the motor voltage to compensate for the velocity-dependent motor back-EMF.

Use setPwmGradient(pwm\_amplitude) to change. pwm\_amplitude range: 0-255

In voltage control mode, increase the PWM gradient to increase the motor current if it decreases too much when the motor increases velocity.

In current control mode, the pwm gradient value is used for initialization only. The driver will calculate the pwm gradient value automatically.


### Run Current

The run current is used to scale the spinning motor current.

Use setRunCurrent(percent) to change. percent range: 0-100

In voltage control mode, the run current scales the PWM amplitude, but the current setting is not measured and adjusted when changes to the operating conditions occur. Use the PWM offset, the PWM gradient, and the run current all three to adjust the motor current.

In current control mode, setting the run current is the way to adjust the spinning motor current. The driver will measure the current and automatically adjust the voltage to maintain the run current, even with the operating conditions change. The PWM offset and the PWM gradient may be changed to help the automatic tuning procedure, but changing the run current alone is enough to adjust the motor current since the driver will adjust the offset and gradient automatically.


### Standstill Mode

The standstill mode determines how the motor will behave when the driver is commanded to be at zero velocity.

Use setStandstillMode(mode) to change. mode values: NORMAL, FREEWHEELING, STRONG\_BRAKING, BRAKING

1.  NORMAL

    In NORMAL mode, the driver actively holds the motor still using the hold current setting to scale the motor current.

2.  FREEWHEELING

    In FREEWHEELING mode, the motor is free to spin freely when the driver is set to zero velocity.

3.  STRONG\_BRAKING and BRAKING

    When the mode is either BRAKING, or STRONG\_BRAKING, the motor coils will be shorted inside the driver so the motor will tend to stay in one place even though current is not actively being driven into the coils.


### Hold Current

The hold current is used to scale the standstill motor current, based on the standstill mode and the hold delay settings.

Use setHoldCurrent(percent) to change. percent range: 0-100

In voltage control mode, the hold current scales the PWM amplitude, but the current setting is not measured and adjusted when changes to the operating conditions occur. Use the PWM offset and the hold current both to adjust the motor current.

In current control mode, setting the hold current is the way to adjust the stationary motor current. The driver will measure the current and automatically adjust the voltage to maintain the hold current, even with the operating conditions change. The PWM offset may be changed to help the automatic tuning procedure, but changing the hold current alone is enough to adjust the motor current since the driver will adjust the offset automatically.




# Hardware Documentation


## Datasheets

[Datasheets](./datasheet)


## TMC2209 Stepper Driver Integrated Circuit

[Trinamic TMC2209-LA Web Page](https://www.trinamic.com/products/integrated-circuits/details/tmc2209-la)


## SilentStepStick Stepper Driver Board

[Trinamic TMC2209 SilentStepStick Web Page](https://www.trinamic.com/support/eval-kits/details/silentstepstick)

