# ee186_lab3
What is the I2C addresses of the accelerometer and magnetometer? Represent the r/w bit using b. 
Form datasheet: “The accelerometer sensor slave address is 0011001b while magnetic sensor slave address is 0011110b.”
Accelerometer: 0011001b, where the trailing b is the r/w bit (1 = read, 0 = write)
Magnetometer: 0011110b, same convention


What is the sub-address (SUB), and how does it differ from the I2C address?
What it is: a one-byte value you send right after the device address that tells the sensor which internal register you want to read or write.
It’s 8 bits, split into two parts:
Bits 6:0 (7 LSBs): the actual register address.
Bit 7 (MSB): auto-increment enable. If it’s 1, the register address goes up by one after each byte, so you can read several consecutive registers in a single transaction. If it’s 0, you access just that one register.
How it differs from the I2C: the I2C (slave) address is 7 bits and selects which device on the bus responds (accelerometer vs. magnetometer). The sub-address is 8 bits (7 address bits + 1 auto-increment bit) and selects which register inside that device you’re accessing. It’s sent as the byte after the device address, not as part of it.


To enable acceleration data measurement, we need to configure the CTRL1 register by selecting the data rate and setting the enable bits. What is the CTRL1 register address and what value should be written to this register to accomplish this?
CTRL_REG1_A
Type: R/W
HEX: 20
Binary: 010 0000
Default: 00000111

I will configure it to 25Hz (0011), Lpen will be 0 for normal mode, and I will enable Zen, Xen, and Yen. This is what I’m writing to the register: 0011 0111.


To read acceleration data for the X, Y, and Z axes, what are the 8-bit sub-addresses that the transmitter would use to request the data?
OUT_X_L_A (28h), OUT_X_H_A (29h)
OUT_Y_L_A (2Ah), OUT_Y_H_A (2Bh)
OUT_Z_L_A (2Ch), OUT_Z_H_A (2Dh)

Name | Type | Hex | Binary 
OUT_X_L_A | R | 28 | 010 1000 
OUT_X_H_A | R | 29 | 010 1001 
OUT_Y_L_A | R | 2A | 010 1010 
OUT_Y_H_A | R | 2B | 010 1011 
OUT_Z_L_A | R | 2C | 010 1100 
OUT_Z_H_A | R | 2D | 010 1101 
For a normal single-register read, the SUB is just the register’s address.


Last part:
Explain how the arguments to HAL I2C Mem Read() should be changed to accomplish this.

To read all three axes in a single I²C transaction, two parameters need to change. First, the MemAddress argument changes from 0x28 to 0x28 | 0x80, which is 0xA8. The LSM303AGR datasheet specifies that the MSB (bit 7) of the sub-address controls address auto-increment. When it is set to 1, the sensor automatically increments its internal register pointer after each byte it sends, so a single read starting at OUT_X_L_A (0x28) will continue through OUT_X_H_A, OUT_Y_L_A, OUT_Y_H_A, OUT_Z_L_A, and OUT_Z_H_A in order. Without this bit, the pointer stays fixed and the same register would be read six times. Second, the Size argument changes from 1 to 6, telling HAL to keep the I2C transaction open and clock in all six bytes before sending a NACK and a STOP condition. The result is one transaction on the bus instead of six, which ensures all six bytes are sampled at the same point in time rather than across six separate reads.


