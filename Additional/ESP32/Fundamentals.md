#### Buttons
- `BOOT` : Used to put the board into flashing mode. Hold this button while flashing the code. 
- `EN` : The reset button. 
#### Pins
1. Power pins
	- 3.3V
	- 5V (labeled as VIN)
	- GND (ground)
2. GPIO pins (General purpose input output)
	Used to read button presses, tun LEDs, Control relays, read sensors,...
	Some pins are used for UART(serial communication), 12C(sensors, display), SPI(fast devices like SD cards), ADC(analog reading), DAC(analog output), PWM(motor speed control).

Pins save for LEDs and buttons, 
	GPIO 2
	GPIO 4
	GPIO 5
	GPIO 18
	GPIO 19
	GPIO 21
	GPIO 23
	GPIO 25
	GPIO 26
	GPIO 27
	GPIO 32
	GPIO 33

Pins can not output a signal,(only for sensors, analog inputs)
	GPIO 34
	GPIO 35
	GPIO 36
	GPIO 39

Pins that affect boot,
	GPIO 0 : must be high to boot
	GPIO 2 : must be high for normal boot
	GPIO 12 : boot voltage selection
	GPIO 15 : boot config

Default UART pins(used for programming),
	GPIO 1 (TX0)
	GPIO 3 (RX0)