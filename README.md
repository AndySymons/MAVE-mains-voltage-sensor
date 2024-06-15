# MAVE-mains-voltage-sensor

Simple ESPHome project to detect the presence of a mains voltage (230V AC).

# Introdcuution

It seems a simple task -- detect tyhe presence or not of a voltage; however, I found no commercial product or reference to any similar project that performss this task. So I built one.  

In my case, I wanted to remotely monitor the positions of two three-positon switches, but the basic circuit can be used to detect any voltage for any purpose. 

The two switches in my case control a boiler -- one for Hot Water (HW) and one for Central Heating (CH). The three positions are "Manual", "Off", or "Auto". The "Off" state is simply a central position in which neither of the two outer contacts are closed, there is no separate "Off" contact. 
I therefore connect one channel of the detector, to the central pole, to allow me to distinguish between "No power" and "Off": "Off" means there is power to the switch but neither of the"Manual" of "Auto" positions is live. My two switches share one power source, so I needed a total of 5 channels.  

## Schematic 

<schematic to follow> 

1. D1: Since the input is AC, the IN4007 diode is required because the reverse-voltage brekdown of the Optocoupler LED is too low to act as a rectifier.
2. L1: the indicator LED is entirely optional. I put it in as as smple way of confirming that current is flowing when a voltage is applied.
3. R1: the 100kΩ resistor is dimensioned to set the current through the optocoupler LED (and indeed the indicator LED) at 2.3 mV for a 230V input. This is sufficient to reliably turn on the photo-transistor. It is rather low for the indicator LED, but that is not important in my case. If it were, the resistor could easily be reduced by a factor of 10: 10kΩ would give a current of 23mA. The maximum for the optocoupler is 50mA. for a 110V mains supply a resistor of 47kΩ could be used. In fact practically any voltage up 700V (the limit of the diode) could be detected by calculating R = V / I, where I is the desired current, which should be between 0.001 (1mA) and 0.05 (50mA).
4. Q1 input: the cheap PC817 optocoupler is adequate here as no high frequency switching is needed. The input side LED is activated by the 2.3mA current that flows when 230V is applied to the input. In my case, the detected signals have a common Neutral line, but these could of course be separated if needed. 
5. Q1 output: The cathode of the transistor side of the optocoupler is connected to a processor input that uses its internal pullup resistor. yje emitter is connected top trge processor ground (not to be confused with the mains Neutral or Earth!). A current on the input side pulls the input down, so the logic is inverted.   
6. P1: microprocessor. I used a Wemos D1 Mini, which is ESP8266-based. It is cheap and compact. It has six easily-usable inputs. There are ten on the board, labelled A0 and D0 to D8; However, D0 (GPIO16) has no pullup resistor, D3 (GPIO0) and D4 (GPIO2) are used during firmware programming (flash), so should preferably be avoided. That a leaves six: D1, D2, D5, D6, D7, and D8. On my board D8 (GPIO15) did not appear to have a functioning pullup, but that may be a spurious hardware fault? Fortunately, I only needed five inputs. If more channels are required, it is a simple mater to find a processor and dev board that has them, or an input multiplexer (expander) sould be employed. 

## Configuration 
See the accompanying ESPHome configuration file in this repository -- 'mave-mains-votage-sensor.yaml'.  
### Binary sensors 
After the standard board configuration, I first configure the binary sensors linked to the input pins. This is self-explanatory, but note the 'delayed_off' times. When an AC voltage is appied at the input the diode acts as a half-wave rectifier. To keep component count low, I do not smooth this, so the outpocoupler output / processor input is actualy a 50Hz square wave. I 'digitally smooth' this by haveing a delayed_off period that must exceed a 50Hz cycle, i.e. 20mS. Experimentally, I found that short times such as 25mS or even 50mS wer not stable; that is to say while the input remained 'on' the detected switch state flickjered to 'off' every few seconds. I suspect that the ESP8266 is not capable of the processing speed to accurately incorporate a very short delay. However, 100mS doıes work stably amd I am in no hurry!
### Text sensors 
The text sensors in the last section are specific to my use-case. For each of the two switches I am monitioring I deduce the position using a Lambda (C++ code).
- Note 1: In ESPHome (unlike Home Assistant) a sensor cannot contain text, only numbers. A text sensor does map well into Home assistant, though.  
- Note 2: A type error occurs if you try to return a text value directly from the Lambda. It ıs necessary to declare a variable with the correct type, 'std::string' then the text is cast to that type in the assignment statements.    

