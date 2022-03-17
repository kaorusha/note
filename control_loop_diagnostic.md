# Motor Driver Control Loop Diagnostic
Adding diagnostic for simple motor control loop.

## hardware
raspberry pi pico has 3 ADC port (GP26,GP27, GP28) with high resolution 12 bit
raspberry pi has only digital pins, need external ADC chip like MCP3008 wired shown [here](https://pimylifeup.com/raspberry-pi-adc/) or other ADC pi board, both additional chip providing 8 ADC channel.

## Log frameworks
* [several log libraries](https://stackoverflow.com/questions/52357/what-is-the-point-of-clog)
### For embedded systems
* [EasyLogger](https://github.com/armink/EasyLogger)
* [log.c](https://github.com/rxi/log.c)
* [McuLog](https://mcuoneclipse.com/2020/06/01/mculog-logging-framework-for-small-embedded-microcontroller-systems/): This article introduces Logging Framework for small Embedded Microcontroller Systems, the code [repo](https://github.com/ErichStyger/McuOnEclipseLibrary). It links shows other similar libraries.
* [ulog](https://github.com/rdpoor/ulog) 
* [NetBurner/RealTimeDataLog](https://github.com/NetBurner/RealTimeDataLog): Used for this company's IOT products (Arm base)
