# MAVE - A mains voltage sensor

A simple ESPHome project to detect the presence of a mains voltage (230V AC).

_N.B: Uses dangerous mains voltages. Do not attempt unless you are 100% sure you know what you are doing!_

## Introduction

It seems a simple task - detect the presence or not of a voltage; however, I found no commercial product or reference to any similar project that performs this task. So I designed and built my own.  

In my case, I wanted to remotely monitor the positions of two three-positon switches, but the same circuit can be used to detect any high AC or DC voltage for any purpose. 

The two switches in my case control a boiler - one for Hot Water (HW) and one for Central Heating (CH). The three positions are "Manual", "Off", or "Auto". The "Off" state is simply a central position in which neither of the two outer contacts are closed, there is no separate "Off" contact. 
I therefore connect one channel of the detector to the central pole, to allow me to distinguish between "No power" and "Off": "Off" means there is power to the switch but neither of the "Manual" of "Auto" positions is live. My two switches share one power source, so I needed a total of 5 channels.  

## Schematic 

-- figure to follow --  

### High voltage side 
  
1. D1: Since the input is AC, the IN4007 diode is required because the reverse-voltage breakdown of the Optocoupler LED is too low to act as a rectifier. For a DC application, this would still be useful protection against polarity reversal. 
2. L1: The indicator LED is entirely optional. I put it in as as smple way of confirming that current is flowing when a voltage is applied.
3. R1: The 100kΩ resistor is dimensioned to set the current through the optocoupler LED (and indeed the indicator LED) at 2.3 mV for a 230V input. This is sufficient to reliably turn on the photo-transistor. It is rather low for the indicator LED, but it being dim is not important in my case. If it were, the resistor could easily be reduced by a factor of 10: 10kΩ would give a current of 23mA. The maximum for the optocoupler is 50mA. For a 110V mains supply, a resistor of 47kΩ could be used. In fact practically any voltage up to 700V (the limit of the diode) could be detected by calculating R = V / I, where I is the desired current, which should be between 0.001 (1mA) and 0.05 (50mA). 
4. Q1 input: the cheap PC817 optocoupler is adequate here as no high frequency switching is needed. The input side LED is activated by the 2.3mA current that flows when 230V is applied to the input. In my case, the detected signals have a common Neutral line, but these could be separated too if required. 
5. Q1 output: The cathode of the transistor side of the optocoupler is connected to a processor input that has an internal pullup resistor. The emitter is connected to the processor Ground (not to be confused with the mains Neutral or Earth!). A current on the input side pulls the input down, so the logic is inverted.

This circuit is repeated identically for the number of detector channels required. Mine has five. 

### Microprocessor

I used a Wemos D1 Mini, which is ESP8266-based. It is cheap and compact. It has five readily-usable inputs - not limited by boot-time conditions and having internal pullup resistors: D1, D2, D5, D6, and D7. Fortunately, five inputs is exactly what I needed! 

There are ten on the board, labelled A0 and D0 to D8. However, A0 is connected to an ADC and does not have an internal pullup resistor; D0 (GPIO16) and D8 (GPIO15) have no pullup resistor; D8 has a pull-down and must be low at boot time; D3 (GPIO0) and D4 (GPIO2) must be high during firmware programming (flash). At a push, some of those could be used provided that the boot and flash time rules are met, and external pullup resistors provided. For me, if more channels were required, I would find a processor and dev board that has them, or employ an I2C I/O expander such as the 16-channel MCP23017.

## Power supply 
For the Wemos D1 mini, a 5V power supply is most convenient. The board itself then regulates the 3V3 supply actually used by te microprocessor. 

I cannibalised a USB charger adapter, removed the USB socket and soldered wires to it for input and output. 

There are however, plenty of cheap 5V power supply modules available! The power demand is under 100mA, so practically any will do.

## Physical build

I built MAVE pn a prototype board with a pre-printed pattern suitable for connecting a DIP socket for the optocouplers, and the development board via straight headers. The optocouplers could be soldered directly, but I find it convenient to be able to do circuit testing before inserting them. Isolation between the mains and low voltage areas are particularly inportant on this board! 

The board fits inside a standard 25mm double pattress box with a blank front plate. These are common and cheap in the UK, visually compatible with the environment in which MAVE is to be used, and plastic, so there is no interference with the WiFi signal.     

## Configuration 
_See the accompanying ESPHome configuration file in this repository: 'mave-mains-votage-sensor.yaml'_

### Binary sensors 

After the standard board configuration part, I first configure the binary sensors linked to the input pins. This is mostly self-explanatory, but note the 'delayed_off' times. When an AC voltage is applied at the input the diode acts as a half-wave rectifier. To keep component count low, I do not smooth this, so the outpocoupler output / processor input is actualy a 50Hz square wave. I 'digitally smooth' this by introducing a delayed_off period that must exceed a 50Hz cycle, i.e. 20mS. Experimentally, I found that short times such as 25mS or even 50mS wer not stable; that is to say while the input remained 'on' the detected switch state flickered to 'off' every few seconds. I suspect that the ESP8266 is not capable of the processing speed to accurately incorporate a very short delay. However, 100mS doıes work stably amd I am in no hurry!

Note: there cannot be any 'delayed_on' time over 20mS as the input would then never turn on. I do not bother with 'anti-bouncing' because my text sensors (below) only refresh every second anyway. Anti-bouncing could be implemented digitally at the HA level, or capacitors could be introduced to smooth the AC input. 

### Text sensors 

The text sensors in the last section are specific to my use-case. For each of the two switches I am monitioring I deduce the position using a Lambda (C++ code).
- Note 1: In ESPHome (unlike Home Assistant) a sensor cannot contain text, only numbers. A text sensor does map well into Home assistant, though.  
- Note 2: A type error occurs if you try to return a text value directly from the Lambda. It ıs necessary to declare a variable with the correct type, 'std::string' then the text is cast to that type in the assignment statements.    






---
If you find the ideas in this repository useful, please [Buy Me a Coffee](https://buymeacoffee.com/andysymons)

