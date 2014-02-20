# Rfu JeeLib #


The repo contains a modified version of the JeeLib RFM12B wireless radio driver for use with the Ciseco RFu 328 units V1.x with an RFM12B.
<http://shop.ciseco.co.uk/rf-328-bare-arduino-atmega-328-compatible-micro-board-rfu-328/>

**For the emonTx V2.x and other current OpenEnergyMonitor hardware units use the unmodified JeeLib Library available here: <https://github.com/jcw/jeelib>**

<br>

The RFu is a tiny ATmega328 + RFM12B radio (SRF can also be used) in Xbee form factor. 
<http://openmicros.org/index.php/articles/88-ciseco-product-documentation/236-rfu328-starter-guide> 

**THE STANDARD jeelib LIBRARY SHOULD NOT BE UPLOADED TO THE RFu V1.1** 

If the standard JeeLib is uploaded to the RFu V1.1 it will enter a reset loop and serial upload will stop working. For technical explanation and method to recover from this see below.  

## Changes made the JeeLib library specific to the RFu V1.1 ##

The library was renamed to 'RFu_JeeLib' to avoid confusion with the unmodified 'jeelib'. This also enables both libraries to be installed in Arduino libraries folder. 

### The following changes where made to RF12.cpp ###

RFM12B SS moved to Dig4/PD4 and INT moved to Dig3/INT1:

```cpp
    #defined(CISECO_RFU)
    // CISECO RFU - RFM12B 0 SS: DIG3     INT:DIG2 INT1
    #define RFM_IRQ     3     //DIG3/INT1 also change LINE 598 AND 600 below
    #define SS_DDR      DDRD
    #define SS_PORT     PORTD
    #define SS_BIT      4     // PD4/DIG4

    #define SPI_SS      10    // PB2, pin 16
    #define SPI_MOSI    11    // PB3, pin 17
    #define SPI_MISO    12    // PB4, pin 18
    #define SPI_SCK     13    // PB5, pin 19

    #elif defined(CISECO_RFU)
        if ((nodeid & NODE_ID) != 0)
            attachInterrupt(1, rf12_interrupt, LOW);
        else
            detachInterrupt(1);
```

Set nINT/VDI pin to always be an input:

```cpp 
    rf12_xfer(0x90A2); // nINT,FAST,134kHz,0dBm,-91dBm - set nINT/VDI pin as output to stop it    interfering with RFu reset
```

Explanation: The nINT/VDI pin (pad 11) on the RFM12B is connected to the Atmega328's RST line on the RFu V1.1. This is to allow OTA uploading when using an SRF radio. The nINT/VDI pin is set as input as default, when the RFM12B is initialised using the standard JeeLib library it sets nINT/VDI as output and drives it low resulting in the ATmega328 on the RFu getting stuck in a reset loop. In this state the ATmega328 on the RFu is unable accept a serial upload. The only way to recover from this reset loop is to connect the RFu to an ISP programmer and re-load the Arduino Uno bootloader. - this issue has now been fixed in software by setting nINT/VSI as input (high impedance).  

Thanks to Matt Lloyd for this software fix. 

#Many thanks to the hard work of JCW from jeelabs.org for the RFM12 driver# 

**JeeLib** is an Arduino IDE library for the "ports" on JeeNodes, the  
"RF12" wireless driver library, timers, low-power code, and several interface  
classes. With examples for many sensor & interface boards made by [JeeLabs]

----

Download the zip 

* Unpack the archive and rename the resulting folder to "RFu_JeeLib".
* Move it to the "libraries" folder in your Arduino sketches area.
* Restart the Arduino IDE to make sure it sees the new files.

The wiki for the un modified JeeLib library is at <http://jeelabs.net/projects/cafe/wiki>.






