# uI2C Library

## Description
**uI2C** is a fork of famous [TinyI2C library](https://github.com/technoblogy/tiny-i2c).

**uI2C** set of minimal I2C Master routines for ATtiny processors with a hardware USI.

The main difference between these routines and most other Tiny Wire libraries is that these don't use buffers, so have minimal memory requirements, and don't impose a 32-byte limit on transmissions.

Note that you need to use Spence Konde's ATTiny Core for these routines to work; otherwise you'll get errors such as:

    error: 'PIN_USI_CL' was not declared in this scope
    error: 'PIN_USI_SCL' was not declared in this scope

## Introduction

I've named these routines uI2C for two reasons: to distinguish them from the existing TinyWire libraries, such as the one included in Spence Konde's ATTiny Core, and to emphasise that these routines don't follow the Arduino Wire library naming conventions.

In addition, these routines differ from the Tiny Wire library routines in the following ways:

### Low memory requirements

These routines don't use buffers, reducing their RAM requirements to a couple of bytes. The standard Wire libraries use 32-byte send and receive buffers requiring at least 32 bytes, which isn't such a problem on the ATmega chips, but on an ATtiny85 this is a significant part of the available RAM.

I've always been puzzled about why the standard wire libraries use 32-byte send and receive buffers, and I haven't been able to find an answer to this on the web. As far as I can see there's no need for buffering as the I2C protocol incorporates handshaking, using the ACK/NACK pulses.

### Unlimited transmission length

These routines don't impose any limit on the length of transmissions. The standard Wire libraries limit the length of any transmission to 32 bytes. This isn't a problem with many I2C applications, such as reading the temperature from a sensor, but it is a problem with applications such as driving an I2C OLED display, which requires you to send 1024 bytes to update the whole display.

### Flexible read

These routines allow you to specify in advance how many bytes you want to read from an I2C peripheral, or you can leave this open-ended and mark the last byte read. This is an advantage when you don't know in advance how many bytes you are going to want to read.

## Compatibility

Although so far I've only tested these routines on a couple of ATtiny chips, they should support all ATtiny chips with the USI peripheral, namely:

* ATTinyX4
* ATTinyX5
* ATTinyX61
* ATTinyX313
* ATTiny1634

These routines are based on the code described by Atmel Application Note AVR310 "Using the USI module as a TWI Master".

## Description

Here's a description of the uI2C routines:

### uI2C.start(address, type)

Starts a transaction with the slave device with the specified address, and specifies if the transaction is going to be a read or a write. It returns a true/false value to say whether the start was successful.

The **type** parameter can have the following values:

* 0: Write to the device.
* 1 to 32767: Read from the device. The number specifies how many reads you are going to do.
* -1: Read an unspecified number of bytes from the device.

If **type** is specified as -1 you must identify the last read by calling **uI2C.readLast()** rather than **uI2C.read()**.

### uI2C.write(data)

Writes a byte of data to a slave device. It returns a true/false value to say whether the write was successful.

### uI2C.read()

Returns the result of reading from a slave device.

### uI2C.readLast()

Returns the result of reading from a slave device and tells the slave to stop sending. You only need to use **uI2C.readlast()** if you called **uI2C.start()** or **uI2C.restart()** with **type** set to -1.

### uI2C.restart(address, type);

Does a restart. The **type** parameter is the same as for **uI2C.start()**.

### uI2C.stop()

Ends the transaction.

## Examples

### Writing to a slave

Writing to a slave is straightforward: for example, to write one byte:

    uI2C.start(Address, 0);
    uI2C.write(byte);
    uI2C.stop();

### Reading from a slave

The uI2C routines allow you to identify the last byte read from a slave in either of two ways:

You can specify the total number of bytes you are going to read, as the second parameter of **uI2C.start()**. With this approach **uI2C.read()** will automatically terminate the last call with a NAK:

    uI2C.start(Address, 2);
    int mins = uI2C.read();
    int hrs = uI2C.read();
    uI2C.stop();

Alternatively you can just specify the second parameter of **uI2C.start()** as -1, and explicitly identify the last **uI2C.read** command by calling **uI2C.readLast()**:

    uI2C.start(Address, -1);
    int mins = uI2C.read();
    int hrs = uI2C.readLast();
    uI2C.stop();

### Writing and reading

Many I2C devices require you to write one or more bytes before reading, to specify the register you want to read from; the read should be introduced with an **uI2C.restart()** call; for example:

    uI2C.start(Address, 0);
    uI2C.write(1);
    uI2C.restart(Address, 2);
    int mins = uI2C.read();
    int hrs = uI2C.read();
    uI2C.stop();
