# The Cube Module Overview

![](../.gitbook/assets/cube-overview.jpg)

### The Cube Fixed Board

* Black, Green, Blue, Purple STM32F427; flash 2MB, RAM 256KB.
* Yellow STM32F777; flash 2MB, Ram 512KB.
* Orange STM32F427; flash 2MB, RAM 1MB.
* On-board 16KB SPI FRAM
* MPU9250 or ICM 20649 integrated accelerometer / gyro.
* MS5611 Barometer
* All sensors connected via SPI.
* Micro SD interfaces via SDIO.

### Vibration Damped IMU board version 1 \(Fitted to Cube Black, Cube Blue, Cube Green\)

* LSM303D integrated accelerometer / magnetometer.
* L3GD20 gyro.
* MPU9250 Gyro / Accel
* MS5611 Barometer
* All sensors connected via SPI.

### Vibration Damped IMU board version 2 \(Fitted to Cube Orange and yellow\)

* ICM20602
* ICM 20948
* MS5611 Barometer
* All sensors connected via SPI.

### I/O ports

* 14 PWM servo outputs \(8 from IO, 6 from FMU\).
* R/C inputs for CPPM, Spektrum / DSM and S.Bus.
* Analogue / PWM RSSI input.
* S.Bus servo output.
* 5 general purpose serial ports, 2 with full flow control
* Two I2C ports
* One SPI port \(un-buffered, for short cables only not recommended for use\).
* Two CAN Bus interface. \(CAN FD for Orange post Beta version\)
* 3 Analogue inputs
* High-powered piezo buzzer driver. \(On expansion board\)
* High-power RGB LED. \(I2C driver compatible Connected externally only\)
* Safety switch / LED.

## System architecture

The Cube continues with the FMU + IO architecture from the previous generation, incorporating the two functional blocks in a single physical module.

![](../.gitbook/assets/cube-system-architecture.png)

### PWM Outputs

The Cube has eight PWM outputs that are connected to IO and can be controlled by IO directly via R/C input and on-board mixing even if FMU is not active \(failsafe / manual mode\). Multiple update rates can be supported on these outputs in three groups; one group of four and two groups of two. PWM signal rates up to 400Hz can be supported. These 8 PWM's are output ONLY and are capable of driving up to 50mA each, but only a total of 100mA for the 8.

Six PWM outputs are connected to FMU and feature reduced update latency. These outputs cannot be controlled by IO in failsafe conditions. Multiple update rates can be supported on these outputs in two groups; one group of four and one group of two.

PWM signal rates up to 400Hz can be supported.

All PWM outputs are EDS-protected, and they are designed to survive accidental mis-connection of servos without being damaged. The servo drivers are specified to drive a 50pF servo input load over 2m of 26AWG servo cable.

the I/O PWM outputs can also be configured as individual GPIOs. Note that these are not high-power outputs – the PWM drivers are designed for driving servos and similar logic inputs only, not relays or LEDs.

### Peripheral Ports

All peripherals are connected through a single 80 pin connector, and the peripherals are connected via a baseboard that can be customized for each application

### Base Board

The initial base board features separate connectors for each of the peripheral ports \(with a few exceptions.

Five serial ports are provided. Serial 1 and 2 feature full flow control. Serial 3 is recommended as the GPS port and has the safety button and \(possibly the safety led\) as well as I2C for the compass and RGB LED. Serial 4 also has I2C, but on the second bus, thus allowing two compass modules to be connected at the same time. Serial 5 is available as a header underneath the board. Serial ports are 3.3V CMOS logic level, 5V tolerant, buffered and ESD-protected.

The SPI port is not buffered; it should only be used with short cable runs. Signals are 3.3V CMOS logic level, but 5V tolerant. SPI is only available to test points on the first base board, along with a CS and INT pin.

Analogue 1-3 are protected against inputs up to 12V, but scaled for 0-3.3V inputs. The RSSI input supports either PWM or analogue RSSI. This input shares a pin with S.Bus output - only one may be connected at a time.

CPPM, S.Bus and DSM/Spektrum input are unchanged from previous versions.

The CAN ports are standard CAN-Bus; termination for one end of the bus is fixed on- board. Drivers are on-board the FMU

The piezo port will drive most piezo elements in the 5 - 300nF range at up to 35V. it is intended to be _extremely loud_, with the achievable sound pressure level limited by the sensitivity of the piezo element being driven.

I2C is direct driven, un-buffered, and pulled up to **\*3.3v on-board\*** the FMU

Serial 5 is used for the on-board ADSB-IN receiver that is featured on newer carrier boards

### Sensors

All flight sensors in The Cube are connected via SPI.

<table>
  <thead>
    <tr>
      <th style="text-align:left">CUBE TYPE</th>
      <th style="text-align:left">IMU1</th>
      <th style="text-align:left">IMU2</th>
      <th style="text-align:left">IMU3</th>
      <th style="text-align:left">Baro1</th>
      <th style="text-align:left">Baro2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Cube Black Blue, Green</td>
      <td style="text-align:left">MPU9250</td>
      <td style="text-align:left">
        <p>LSM303D</p>
        <p>/L3GD20</p>
      </td>
      <td style="text-align:left">MPU9250</td>
      <td style="text-align:left">MS5611</td>
      <td style="text-align:left">MS6511</td>
    </tr>
    <tr>
      <td style="text-align:left">Cube Purple</td>
      <td style="text-align:left">MPU9250</td>
      <td style="text-align:left">NA</td>
      <td style="text-align:left">NA</td>
      <td style="text-align:left">MS5611</td>
      <td style="text-align:left">NA</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>Cube Orange</p>
        <p>/Yellow</p>
      </td>
      <td style="text-align:left">ICM20948</td>
      <td style="text-align:left">ICM20602</td>
      <td style="text-align:left">ICM20649</td>
      <td style="text-align:left">MS5611</td>
      <td style="text-align:left">MS5611</td>
    </tr>
  </tbody>
</table>IMU1, Non-Isolated   
IMU2 & 3, Isolated  
Data-ready signals from all sensors are \***NOT ROUTED on the Isolated IMU\***

## Power Architecture

The Cube removes the power management from the FMU, the Servo rail is no longer the primary source of backup power for the FMU, and it leaves it there for the IO last chance failsafe.

The supply of 3.3v

* Split digital and analogue power domains for FMU and sensors.
* Backup power for IO in the case of FMU power supply failure.

### Power management module \(separate from the FMU\)

Key features of The Cube power architecture:

* Single, independent 5V supply for the flight controller and peripherals.
* Integration with **\*2 power bricks\*** or compatible alternative, including current and voltage sensing.
* Low power consumption and heat dissipation.
* Power distribution and monitoring for peripheral devices.
* Protection against common wiring faults; under/over-voltage protection, overcurrent protection, thermal protection.
* Brown-out resilience and detection.

### FMU and IO Power Supplies

Both FMU and IO operate at 3.3V, and each has its own private dual-channel regulator. As in Pixhawk v1, each regulator features a power-on reset output tied to the regulator’s internal power-up and drop-out sequencing.

### Power Sources

Power may be supplied to The Cube via USB, via the power brick port, or the second brick port. Each power source is protected against reverse-polarity connections and back-powering from other sources.

The FMU + IO power budget is 250mA, including all LEDs and the Piezo buzzer. Peripheral power is limited to 2.5A total.

USB IS NOT RECOMMENDED IN FLIGHT ON Nuttx code

### Power Brick Port

The brick port is the preferred power source for Cube, and brick power will always be selected if it is available.

### Servo Power

The Cube supports both standard \(5V\) and high-voltage \(up to 10V\) servo power with some restrictions.

IO will accept power from the servo connector up to 10V. This allows IO to failover to servo power in all cases if the main power supply is lost or interrupted.

**\*FMU and peripherals will NOT accept power from the servo connector.\***

### Aux Power

The Cube introduces a backup power port; this is set up the same as the primary power input.

At input voltages over 5.7V power is locked out.

The Cube and peripherals combined may draw up to 2.75A total when operating on Aux power, provided that the Brick or other power source can supply the required current.

Power is never supplied by The Cube to servos.

### Servo rail

The I/O chip takes power up to 10.5v from the servo rail; this is used to revert to manual mode in the unfortunate event that the other two main sources of power fail. This is only useful for plane, and only useful if the I/O chip has been mapped correctly.

### USB Power

Power from USB is supported for software update, testing and development purposes. USB power is supplied to the peripheral ports for testing purposes, however total current consumption must typically be limited to 500mA, including peripherals, to avoid overloading the host USB port.

### Multiple Power Sources

When more than one power source is connected, power will be drawn from the highest-priority source with a valid input voltage.

In most cases, FMU should be powered via the power brick or a compatible off board regulator via the brick port or auxiliary power rail.

In desktop testing scenarios, taking power from USB avoids the need for a BEC or similar servo power source \(though servos themselves will still need external power\).

### Summary

For each of the components listed, the input voltage ranges over which the device can be powered from each input is shown.

<table>
  <thead>
    <tr>
      <th style="text-align:left"></th>
      <th style="text-align:left"><b>Brick port</b>
      </th>
      <th style="text-align:left"><b>Aux port</b>
      </th>
      <th style="text-align:left"><b>USB port</b>
      </th>
      <th style="text-align:left"><b>Servo rail</b>
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">FMU</td>
      <td style="text-align:left">4 - 5.7V</td>
      <td style="text-align:left">4 - 5.7V</td>
      <td style="text-align:left">4 - 5.7V</td>
      <td style="text-align:left">NIL</td>
    </tr>
    <tr>
      <td style="text-align:left">IO</td>
      <td style="text-align:left">4 - 5.7V</td>
      <td style="text-align:left">4 &#x2013; 5.7V</td>
      <td style="text-align:left">4 - 5.7V</td>
      <td style="text-align:left">4-10.5V</td>
    </tr>
    <tr>
      <td style="text-align:left">Peripherals</td>
      <td style="text-align:left">
        <p>4 - 5.7V,</p>
        <p>2.5A max</p>
      </td>
      <td style="text-align:left">
        <p>4 - 5.7V</p>
        <p>2.5A max</p>
      </td>
      <td style="text-align:left">
        <p>4 - 5.7V</p>
        <p>250mA max</p>
      </td>
      <td style="text-align:left">NIL</td>
    </tr>
  </tbody>
</table>### Peripherals

**Peripheral Power \(on power module\)**

The Cube provides power routing, over/under voltage detection and protection, filtering, switching, current-limiting and transient suppression for peripherals. Power outputs to peripherals feature ESD and EMI filtering, and the power supply protection scheme ensures that no more than 5.5V is presented to peripheral devices. Power is disconnected from the peripherals when the available supply voltage falls below 2.7V, or rises above approximately 5.7V.

Peripheral power is split into two groups:

* Serial 1 has a private 1.5A current limit, intended for powering a low power

telemetry radio. This output is separately EMI filtered and draws directly from the USB / Brick inputs. Peak power draw on this port should not exceed 2A, which should be sufficient for a 30dBm transmitter of reasonable efficiency.

* All other peripherals share a 1A current limit and a single power switch. Peak power draw on this port should not exceed 1.5A.

Each group is individually switched under software control.

The Spektrum / DSM R/C interface draws power **\*from its own regulator\***, rather than from either of the groups above. This port is switched under software control so that Spektrum / DSM binding can be implemented. Spektrum receivers generally draw

~25mA.

s.Bus and CPPM receivers are powered by a dedicated power supply. Please do not connect any servos to this power, only an RX by itself.

**Capacitor Backup**

Both the FMU and IO microcontrollers feature Capacitor-backed real-time clocks and SRAM. The on-board backup Capacitor has capacity sufficient for the intended use of the clock and SRAM, which is to provide storage to permit orderly recovery from unintended power loss or other causes of in-air restarts.

The capacitors are recharged from the FMU 3.3V rail.

this will only function in the event of software existing to support this feature.

**Voltage, Current and Fault Sensing**

The battery voltage and current reported **\*by both bricks\*** can be measured by the FMU. In addition, the 5V unregulated supply rail can be measured \(to detect brown- out conditions\). IO can measure the servo power rail voltage. Over-current conditions on the peripheral power ports can be detected by the FMU. Hardware lock-out prevents damage due to persistent short-circuits on these ports. The lock- out can be reset by FMU software.

The under/over voltage supervisor for FMU provides an output that is used to hold FMU in reset during brown-out events.

### EMI Filtering and Transient Protection \(on the normal Base Board, must be specified for externally supplied base boards.\)

EMI filtering is provided at key points in the system using high-insertion-loss pass- through filters. These filters are paired with TVS diodes at the peripheral connectors to suppress power transients.

Reverse polarity protection is provided at each of the power inputs.

USB signals are filtered and terminated with a combined termination/TVS array. Most digital peripheral signals \(all PWM outputs, serial ports, I2C port\) are driven using ESD-enhanced buffers and feature series blocking resistors to reduce the risk of damage due to transients or accidental misconnections.

## The Cube Series Interface Spec

_Scope of this Document_

_This document covers the complete interface standard and core mechanical, electrical and external connection options of The Cube module series. Sections marked as LT \(long term\) are intended to be kept stable to isolate vehicle from autopilot revisions._

**Interface Standard**

_**Connector Series**_

* Low density: 0.1” over mould Futaba keyed servo connectors \(Mfg. to be identified\)
  * Cabling: AWG24, ribbon or round, iconic colour scheme
* Stack: [Hirose DF17,](http://www.google.ch/url?sa=t&amp;rct=j&amp;q&amp;esrc=s&amp;source=web&amp;cd=5&amp;cad=rja&amp;uact=8&amp;ved=0CEoQFjAE&amp;url=http%3A%2F%2Fwww.hirose.co.jp%2Fcataloge_hp%2Fen_DF17_20130411.pdf&amp;ei=ya8eU_qbCoer7AbVjYD4Cw&amp;usg=AFQjCNEe_OW87i6iomdn0UDTl6d5UkUhiw&amp;sig2=00Z8iCRQyAbvbU_T63wEeQ&amp;bvm=bv.62788935%2Cd.ZGU) [80pos,](http://www.google.ch/url?sa=t&amp;rct=j&amp;q&amp;esrc=s&amp;source=web&amp;cd=5&amp;cad=rja&amp;uact=8&amp;ved=0CEoQFjAE&amp;url=http%3A%2F%2Fwww.hirose.co.jp%2Fcataloge_hp%2Fen_DF17_20130411.pdf&amp;ei=ya8eU_qbCoer7AbVjYD4Cw&amp;usg=AFQjCNEe_OW87i6iomdn0UDTl6d5UkUhiw&amp;sig2=00Z8iCRQyAbvbU_T63wEeQ&amp;bvm=bv.62788935%2Cd.ZGU) 4 mm stacking height, 0.5 mm pitch, drop-proof
* High density: [JST-GH](http://www.jst-mfg.com/product/detail_e.php?series=105) 1.25 mm
* Cabling: AWG28, ribbon, iconic colour scheme
* Power Module: [Molex Clik-Mate](http://www.molex.com/molex/products/family?channel=products&amp;chanName=family&amp;key=clikmate_wiretoboard_connectors) 2 mm for both main and backup power \( on bottom of board?\)

**The Cube**

Mechanical: 30x30 mm M3 mounting hole pattern, 35x35 mm footprint 80 position DF17 connector. Carries _all_ autopilot interface connections.

* Minimal \(read: really minimal\) electrical protection
* No power management
* 3.8 to 5.7V operation \(absolute maximum ratings\)
* 4.0 to 5.5V operation \(compliant rating\)

**The Cube IO**

Total connectivity

* I2C2
* 2x CAN: CAN1 and CAN2
* 4x UART: TELEM1, TELEM2, GPS \(I2C 1 embedded\), SERIAL4\(I2C 2 embedded\)
* 1x Console: CONSOLE \(SERIAL5\)
* 1x HMI: USB extender

**Power 6 pos \(ClikMate 6 pos 2.0mm\)**

<table>
  <thead>
    <tr>
      <th style="text-align:left"><b>Pin #</b>
      </th>
      <th style="text-align:left"><b>Name</b>
      </th>
      <th style="text-align:left"><b>Dir</b>
      </th>
      <th style="text-align:left"><b>Wire Color</b>
      </th>
      <th style="text-align:left"><b>Description</b>
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">1</td>
      <td style="text-align:left">VDD 5V Brick</td>
      <td style="text-align:left">in</td>
      <td style="text-align:left">red / gray</td>
      <td style="text-align:left">Supply from Brick to AP</td>
    </tr>
    <tr>
      <td style="text-align:left">2</td>
      <td style="text-align:left">VDD 5V Brick</td>
      <td style="text-align:left">in</td>
      <td style="text-align:left">red / gray</td>
      <td style="text-align:left">Supply from Brick to AP</td>
    </tr>
    <tr>
      <td style="text-align:left">3</td>
      <td style="text-align:left">
        <p>BATT_VOLTAGE</p>
        <p>_SENS_PROT</p>
      </td>
      <td style="text-align:left">in</td>
      <td style="text-align:left">black</td>
      <td style="text-align:left">Battery voltage connector</td>
    </tr>
    <tr>
      <td style="text-align:left">4</td>
      <td style="text-align:left">BATT_CURRENT_SENS_PROT</td>
      <td style="text-align:left">in</td>
      <td style="text-align:left">black</td>
      <td style="text-align:left">Battery current connector</td>
    </tr>
  </tbody>
</table>|  |  |  |  |  |
| :--- | :--- | :--- | :--- | :--- |
| 5 | GND | - | black | GND connection |
| 6 | GND | - | black | GND connection |

**Backup Power 6 pos**

| **Pin \#** | **Name** | **Dir** | **Wire Color** | **Description** |
| :--- | :--- | :--- | :--- | :--- |
| 1 | VDD 5V Brick | in | red / gray | Supply from Brick to AP |
| 2 | VDD 5V Brick | in | red / gray | Supply from Brick to AP |
| 3 | AUX\_BATT\_VOL TAGE\_SENS |  |  | Aux Battery voltage connector |
| 4 | AUX\_BATT\_CUR RENT\_SENS | in | black | Aux Battery current connector |
| 5 | GND | - | black | GND connection |
| 6 | GND | - | black | GND connection |

**I2C - 4 pos \(1 fitted as a stand alone, I2C\_2, \(old internal\)**

1. _connector: I2C2 bus_

| **Pin \#** | **Name** | **Dir** | **Wire Color** | **Description** |
| :--- | :--- | :--- | :--- | :--- |
| 1 | VCC\_5V | out | red / gray | Supply to peripheral from AP |
| 2 | SCL | in/out | blue / black | SCL, 5V level, pull-up on AP |
| 3 | SDA | in/out | green / black | SDA, 5V level, pull-up on AP |
| 4 | GND | - | black | GND connection |

**CAN \(2 fitted\)**

1. _connectors: CAN1 and CAN2 buses_

| **Pin \#** | **Name** | **Dir** | **Wire Color** | **Description** |
| :--- | :--- | :--- | :--- | :--- |
| 1 | VCC\_5V | out | red / gray | Supply to peripheral from AP |
| 2 | CAN\_H | in/out | yellow / black | 12V |
| 3 | CAN\_L | in/out | green / black | 12V |
| 4 | GND | - | black | GND connection |

**UART GENERIC \(autopilot side\)**

_2 connectors: TELEM1, TELEM2_

| **Pin \#** | **Name** | **Dir** | **Wire Color** | **Description** |
| :--- | :--- | :--- | :--- | :--- |
| 1 | VCC\_5V | out | red / gray | Supply to GPS from AP |
| 2 | MCU\_TX | out | yellow / black | 3.3V-5.0V TTL level, TX of AP |
| 3 | MCU\_RX | in | green / black | 3.3V-5.0V TTL level, RX of AP |
| 4 | MCU\_CTS \(TX\) | out | gray / black | 3.3V-5.0V TTL level or TX of AP |
| 5 | MCU\_RTS \(RX\) | in | gray / black | 3.3V-5.0V TTL level or RX of AP |
| 6 | GND | - | black | GND connection |

**UART GPS \(autopilot side, I2C is the original “External” bus\)**

_1 connector: GPS_

| **Pin \#** | **Name** | **Dir** | **Wire Color** | **Description** |
| :--- | :--- | :--- | :--- | :--- |
| 1 | VCC\_5V | in | red | Supply to GPS from AP |
| 2 | GPS\_RX | in | black | 3.3V-5.0V TTL level, TX of AP |
| 3 | GPS\_TX | out | black | 3.3V-5.0V TTL level, RX of AP |
| 4 | SCL | in | black | 3.3V-5.0V I2C1 |
| 5 | SDA | in/out | black | 3.3V-5.0V I2C1 |
| 6 | BUTTON | out | black | Signal shorted to GND on press |
| 7 | BUTTON\_LED | out | black | LED Driver for Safety Button |
| 8 | GND | - | black | GND connection |

**UART 4 \(I2C 2, the original “Internal” bus\)**

_1 connector: GPS_

| **Pin \#** | **Name** | **Dir** | **Wire Color** | **Description** |
| :--- | :--- | :--- | :--- | :--- |
| 1 | VCC\_5V | out | red / gray | Supply to GPS from AP |
| 2 | MCU\_TX | out | yellow / black | 3.3V-5.0V TTL level, TX of AP |

|  |  |  |  |  |
| :--- | :--- | :--- | :--- | :--- |
| 3 | MCU\_RX | in | green / black | 3.3V-5.0V TTL level, RX of AP |
| 4 | SCL | out | gray / black | 3.3V-5.0V I2C2 |
| 5 | SDA | in | gray / black | 3.3V-5.0V I2C2 |
| 6 | GND | - | black | GND connection |

**UART 5\(Debug\) and S.Bus out**

Original Carrier board

| **Pin \#** | **Name** | **Dir** | **Wire Color** | **Description** |
| :--- | :--- | :--- | :--- | :--- |
| 1 | S.Bus Out | out |  | 3.3V-5.0V TTL level, TX of AP |
| 2 | MCU\_TX | out |  | 3.3V-5.0V TTL level, TX of AP |
| 3 | VDD\_Servo | OUT |  | Servo rail voltage |
| 4 | MCU\_RX | in |  | 3.3V-5.0V TTL level, RX of AP |
| 5 | GND | out |  | GND |
| 6 | GND | out |  | GND |

**S.Bus out**

New ADSB Carrier board

| **Pin \#** | **Name** | **Dir** | **Wire Color** | **Description** |
| :--- | :--- | :--- | :--- | :--- |
| 1 | S.Bus Out | out |  | 3.3V-5.0V TTL level, TX of AP |
| 2 |  |  |  |  |
| 3 | VDD\_Servo | OUT |  | Servo rail voltage |
| 4 |  |  |  |  |
| 5 | GND | out |  | GND |
| 6 | GND | out |  | GND |

**Debug \( New Standard Debug\) \(Digikey PN for housing SM06B-SURS-TF\(LF\)\(SN\)-ND\)**

IO DEBUG

<table>
  <thead>
    <tr>
      <th style="text-align:left"><b>Pin #</b>
      </th>
      <th style="text-align:left"><b>Name</b>
      </th>
      <th style="text-align:left"><b>Dir</b>
      </th>
      <th style="text-align:left"><b>Wire Color</b>
      </th>
      <th style="text-align:left"><b>Description</b>
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">1</td>
      <td style="text-align:left">VDD 5V PEIPH</td>
      <td style="text-align:left">OUT</td>
      <td style="text-align:left"></td>
      <td style="text-align:left">5V</td>
    </tr>
    <tr>
      <td style="text-align:left">2</td>
      <td style="text-align:left">IO_TX</td>
      <td style="text-align:left">out</td>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <p>3.3V-5.0V TTL level, TX of AP</p>
        <p>IO_uart1 TX</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">3</td>
      <td style="text-align:left">IO_RX</td>
      <td style="text-align:left">in</td>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <p>3.3V-5.0V TTL level, RX of AP</p>
        <p>IO_uart1 RX</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">4</td>
      <td style="text-align:left">IO-SWDIO</td>
      <td style="text-align:left">I/O</td>
      <td style="text-align:left"></td>
      <td style="text-align:left">Serial wire debug I/O</td>
    </tr>
    <tr>
      <td style="text-align:left">5</td>
      <td style="text-align:left">IO-SWCLK</td>
      <td style="text-align:left">I/O</td>
      <td style="text-align:left"></td>
      <td style="text-align:left">Serial wire Clock</td>
    </tr>
    <tr>
      <td style="text-align:left">6</td>
      <td style="text-align:left">GND</td>
      <td style="text-align:left">out</td>
      <td style="text-align:left"></td>
      <td style="text-align:left">GND</td>
    </tr>
  </tbody>
</table>FMU DEBUG

<table>
  <thead>
    <tr>
      <th style="text-align:left"><b>Pin #</b>
      </th>
      <th style="text-align:left"><b>Name</b>
      </th>
      <th style="text-align:left"><b>Dir</b>
      </th>
      <th style="text-align:left"><b>Wire Color</b>
      </th>
      <th style="text-align:left"><b>Description</b>
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">1</td>
      <td style="text-align:left">VDD 5V PEIPH</td>
      <td style="text-align:left">OUT</td>
      <td style="text-align:left"></td>
      <td style="text-align:left">5V</td>
    </tr>
    <tr>
      <td style="text-align:left">2</td>
      <td style="text-align:left">FMU_TX (SERIAL 5)</td>
      <td style="text-align:left">out</td>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <p>3.3V-5.0V TTL level, TX of AP</p>
        <p>FMU_uart5 TX</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">3</td>
      <td style="text-align:left">FMU_RX (SERIAL 5)</td>
      <td style="text-align:left">in</td>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <p>3.3V-5.0V TTL level, RX of AP</p>
        <p>FMU_uart5 RX</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">4</td>
      <td style="text-align:left">FMU-SWDIO</td>
      <td style="text-align:left">I/O</td>
      <td style="text-align:left"></td>
      <td style="text-align:left">Serial wire debug I/O</td>
    </tr>
    <tr>
      <td style="text-align:left">5</td>
      <td style="text-align:left">FMU-SWCLK</td>
      <td style="text-align:left">I/O</td>
      <td style="text-align:left"></td>
      <td style="text-align:left">Serial wire Clock</td>
    </tr>
    <tr>
      <td style="text-align:left">6</td>
      <td style="text-align:left">GND</td>
      <td style="text-align:left">out</td>
      <td style="text-align:left"></td>
      <td style="text-align:left">GND</td>
    </tr>
  </tbody>
</table>**Analogue**

| **Pin \#** | **Name** | **Dir** | **Wire Color** | **Description** |
| :--- | :--- | :--- | :--- | :--- |
| 1 | VDD\_5V\_Periph | out |  |  |
| 2 | Pressure sense in | in |  |  |
| 3 | GND | out |  | GND |

**Spektrum**

| **Pin \#** | **Name** | **Dir** | **Wire Color** | **Description** |
| :--- | :--- | :--- | :--- | :--- |
| 1 | VDD\_3v3\_spektru m | out |  | Independent supply 3v3. |
| 2 | IO\_USART1\_RX | in |  |  |
| 3 | GND | out |  | GND |

**HMI \(Buzzer, USB, LEDs\)**

| **Pin \#** | **Name** | **Dir** | **Wire Color** | **Description** |
| :--- | :--- | :--- | :--- | :--- |
| 1 | VCC\_5V | out | red / gray | Supply to GPS from AP |
| 2 | D\_PLUS | in/out | green / black | 3.3V |
| 3 | D\_MINUS | in/out | red / black | 3.3V |
| 4 | GND | - | black | GND connection |
| 5 | BE\_LED | out | black | Boot / Error Led \(FW updates\) |
| 6 | BUZZER | out | gray / black | VBAT \(8.4 - 42V\) |

**Back Edge \( may rearrange to suit PCB layout\)**

_**SERVO HEADER \(0.1”, 1/1/15 power layout\)**_

| **Position** | **Name** | **Dir** | **Wire Color** | **Description** |
| :--- | :--- | :--- | :--- | :--- |
| 15 | RC / SBUS IN | in/out | black | 3.3V \(4..5V powered\) |
| 14 | MAIN\_OUT\_8 | out | black | 3.3V servo signal, servo rail power |
| 13 | MAIN\_OUT\_7 | out | black | 3.3V servo signal, servo rail power |
| 12 | MAIN\_OUT\_6 | out | black | 3.3V servo signal, servo rail power |
| 11 | MAIN\_OUT\_5 | out | black | 3.3V servo signal, servo rail power |
| 10 | MAIN\_OUT\_4 | out | black | 3.3V servo signal, servo rail power |
| 9 | MAIN\_OUT\_3 | out | black | 3.3V servo signal, servo rail power |
| 8 | MAIN\_OUT\_2 | out | black | 3.3V servo signal, servo rail power |

|  |  |  |  |  |
| :--- | :--- | :--- | :--- | :--- |
| 7 | MAIN\_OUT\_1 | out | black | 3.3V servo signal, servo rail power |
| 6 | AUX\_OUT\_6 | out | black | 3.3V servo signal, servo rail power |
| 5 | AUX\_OUT\_5 | out | black | 3.3V servo signal, servo rail power |
| 4 | AUX\_OUT\_4 | out | black | 3.3V servo signal, servo rail power |
| 3 | AUX\_OUT\_3 | out | black | 3.3V servo signal, servo rail power |
| 2 | AUX\_OUT\_2 | out | black | 3.3V servo signal, servo rail power |
| 1 | AUX\_OUT\_1 | out | black | 3.3V servo signal, servo rail power |

**80 pin header \(LONG TERM STANDARD!\)**

| **Pin \#** | **Name** | **Dir** | **Description** |
| :--- | :--- | :--- | :--- |
| 1 | FMU-SWDIO | i/o | Single wire debug io |
| 2 | !FMU- LED\_AMBER | o | Boot error LED \( drive only, use Fet to control led\) |
| 3 | FMU-SWCLK | o | single wire debug clock |
| 4 | I2C\_2\_SDA | i/o | I2C data io |
| 5 | !EXTERN\_CS | o | chip select for external SPI \(NC, just for debugging\) |
| 6 | I2C\_2\_SCL | o | i2c clock |
| 7 | FMU-!RESET | i | reset pin for the FMU |
| 8 |  |  | Future compatibility |
| 9 | VDD\_SERVO\_IN | i | power for last resort i/o failsafe |
| 10 |  |  | Future compatibility |
| 11 | EXTERN\_DRDY | i | interrupt pin for external SPI \(NC, just for debugging\) |
| 12 | SERIAL\_5\_RX | i |  |
| 13 | GND |  | System GND |
| 14 | SERIAL\_5\_TX | o |  |
| 15 | GND |  | System GND |
| 16 | SERIAL\_4\_RX | i |  |
| 17 | SAFETY |  | Safety button input |
| 18 | SERIAL\_4\_TX | o |  |
| 19 | vdd\_3V3\_SPECT RUM\_EN | o | enable for the spectrum voltage regulator |
| 20 | SERIAL\_3\_RX | i |  |
| 21 | PREASSURE\_SE NS\_IN | a i | Analogue port, for pressure sensor, or Laser range finder, or Sonar |
| 22 | SERIAL\_3\_TX | o |  |
| 23 | AUX\_BATT\_VOL TAGE\_SENS | a i | Voltage sense for Aux battery input |
| 24 | ALARM | o | Buzzer PWM signal |
| 25 | AUX\_BATT\_CUR | a i | Current sense for Aux battery input |

<table>
  <thead>
    <tr>
      <th style="text-align:left"></th>
      <th style="text-align:left">RENT_SENS</th>
      <th style="text-align:left"></th>
      <th style="text-align:left"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">26</td>
      <td style="text-align:left">IO-VDD_3V3</td>
      <td style="text-align:left">i</td>
      <td style="text-align:left">IO chip power, pinned through for debug</td>
    </tr>
    <tr>
      <td style="text-align:left">27</td>
      <td style="text-align:left">!VDD_5V_PERIP H_EN</td>
      <td style="text-align:left">o</td>
      <td style="text-align:left">enable signal for Peripherals</td>
    </tr>
    <tr>
      <td style="text-align:left">28</td>
      <td style="text-align:left">!IO- LED_SAFETY_P ROT</td>
      <td style="text-align:left">o</td>
      <td style="text-align:left">IO-LED_SAFETY pinned out for IRIS</td>
    </tr>
    <tr>
      <td style="text-align:left">29</td>
      <td style="text-align:left">VBUS</td>
      <td style="text-align:left">i</td>
      <td style="text-align:left">vbus, voltage from USB plug</td>
    </tr>
    <tr>
      <td style="text-align:left">30</td>
      <td style="text-align:left">SERIAL2_RTS</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">31</td>
      <td style="text-align:left">OTG_DP1</td>
      <td style="text-align:left">i/o</td>
      <td style="text-align:left">DATA P from USB</td>
    </tr>
    <tr>
      <td style="text-align:left">32</td>
      <td style="text-align:left">SERIAL2_CTS</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">33</td>
      <td style="text-align:left">OTG_DM1</td>
      <td style="text-align:left">i/o</td>
      <td style="text-align:left">DATA M from USB</td>
    </tr>
    <tr>
      <td style="text-align:left">34</td>
      <td style="text-align:left">SERIAL2_RX</td>
      <td style="text-align:left">i</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">35</td>
      <td style="text-align:left">I2C_1_SDA</td>
      <td style="text-align:left">i/o</td>
      <td style="text-align:left">I2C data i/o</td>
    </tr>
    <tr>
      <td style="text-align:left">36</td>
      <td style="text-align:left">SERIAL2_TX</td>
      <td style="text-align:left">o</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">37</td>
      <td style="text-align:left">I2C_1_SCL</td>
      <td style="text-align:left">o</td>
      <td style="text-align:left">I2C clock</td>
    </tr>
    <tr>
      <td style="text-align:left">38</td>
      <td style="text-align:left">SERIAL1_RX</td>
      <td style="text-align:left">i</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">39</td>
      <td style="text-align:left">CAN_L_2</td>
      <td style="text-align:left">i/o</td>
      <td style="text-align:left">Canbus Low signal driver on FMU</td>
    </tr>
    <tr>
      <td style="text-align:left">40</td>
      <td style="text-align:left">SERIAL1_TX</td>
      <td style="text-align:left">o</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">41</td>
      <td style="text-align:left">CAN_H_2</td>
      <td style="text-align:left">i/o</td>
      <td style="text-align:left">Canbus High signal driver on FMU</td>
    </tr>
    <tr>
      <td style="text-align:left">42</td>
      <td style="text-align:left">SERIAL1_RTS</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">43</td>
      <td style="text-align:left">!VDD_5V_PERIP H_OC</td>
      <td style="text-align:left">i</td>
      <td style="text-align:left">error state message from Periph power supply</td>
    </tr>
    <tr>
      <td style="text-align:left">44</td>
      <td style="text-align:left">SERIAL1_CTS</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">45</td>
      <td style="text-align:left">!VDD_5V_HIPOW ER_OC</td>
      <td style="text-align:left">i</td>
      <td style="text-align:left">error state message from High power Periph power supply</td>
    </tr>
    <tr>
      <td style="text-align:left">46</td>
      <td style="text-align:left">IO-USART1_TX</td>
      <td style="text-align:left">o</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">47</td>
      <td style="text-align:left">
        <p>BATT_VOLTAGE</p>
        <p>_SENS_PROT</p>
      </td>
      <td style="text-align:left">a i</td>
      <td style="text-align:left">Voltage sense from main battery</td>
    </tr>
    <tr>
      <td style="text-align:left">48</td>
      <td style="text-align:left">IO- USART1_RX_SP ECTRUM_DSM</td>
      <td style="text-align:left">o</td>
      <td style="text-align:left">signal from Spectrum receiver</td>
    </tr>
    <tr>
      <td style="text-align:left">49</td>
      <td style="text-align:left">BATT_CURRENT</td>
      <td style="text-align:left">a i</td>
      <td style="text-align:left">Current sense from main battery</td>
    </tr>
  </tbody>
</table>|  | \_SENS\_PROT |  |  |
| :--- | :--- | :--- | :--- |
| 50 | FMU-CH1-PROT | o |  |
| 51 | SPI\_EXT\_MOSI | o | External SPI, for debug only |
| 52 | FMU-CH2-PROT | o |  |
| 53 | VDD\_SERVO | i | VDD\_Servo, for monitoring servo bus |
| 54 | FMU-CH3-PROT | o |  |
| 55 | !VDD\_BRICK\_VA LID | i | main power valid signal |
| 56 | FMU-CH4-PROT | o |  |
| 57 | !VDD\_BACKUP\_ VALID | i | backup power valid signal |
| 58 | FMU-CH5-PROT | o |  |
| 59 | !VBUS\_VALID | i | USB bus valid signal |
| 60 | FMU-CH6-PROT | o |  |
| 61 | VDD\_5V\_IN | i | main power into FMU from power selection |
| 62 | PPM-SBUS- PROT | i |  |
| 63 | VDD\_5V\_IN | i | main power into FMU from power selection |
| 64 | S.BUS\_OUT | o |  |
| 65 | IO-VDD\_5V5 | o | power to RX |
| 66 | IO-CH8-PROT | o |  |
| 67 | SPI\_EXT\_MISO | i | External SPI, for debug only |
| 68 | IO-CH7-PROT | o |  |
| 69 | IO-SWDIO | i/o | IO single wire debug i/o |
| 70 | IO-CH6-PROT | o |  |
| 71 | IO-SWCLK | o | IO single wire debug clock |
| 72 | IO-CH5-PROT | o |  |
| 73 | SPI\_EXT\_SCK | o | External SPI, for debug only |
| 74 | IO-CH4-PROT | o |  |
| 75 | IO-!RESET | i | IO reset pin |

| 76 | IO-CH3-PROT | o |  |
| :--- | :--- | :--- | :--- |
| 77 | CAN\_L\_1 | i/o | Canbus Low signal driver on FMU |
| 78 | IO-CH2-PROT | o |  |
| 79 | CAN\_H\_1 | i/o | Canbus High signal driver on FMU |
| 80 | IO-CH1-PROT | o |  |

## Differences between Cube colours

![](../.gitbook/assets/cube-series-spec-updated.png)

### List of features The Cube

* Three IMU's
  * these consist of 2 on the IMU board
  * 1 fixed to the FMU
* Two onboard compasses Cube Black, reen, Blue
  * these consist of 1 on the IMU board
  * 1 Fixed on the FMU
* One onboard compass Cube Orange, Cube Yellow
* Two Baros
  * 1 on the IMU \(this Baro will most likely be removed in favour of a dedicated external Barometer.
  * 1 Fixed on the FMU
* Dual Power input
  * This removes the option of redundancy from the Servo rail and replaces it with a dedicated second power plug
  * A dedicated power protection Zener diode and Fet have been added to protect from voltages over 5.6v being applied to Aux input 2
  * This is only on the "PRO" carrier board mini carrier board still draws the backup from the servo rail.
* Dual external I2C
  * This allows for connection of items to either I2C port, potentially allowing two GPS / Mag units to be plugged in without the Mags conflicting.
* Power monitoring pins are now routed to the I/O chip, these will allow for the logging of power events during an inflight reboot.
  * Brick OK, Backup OK, and FMU 3.3V are all connected to a digital pin on the I/O via a 220Ohm resister.

