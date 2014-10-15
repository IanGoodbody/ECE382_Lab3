## Lab 3, LCD Interfacing

#### PreLab

#### Code Analysis

The origional assembly code provided makes four calls to the `#writeNokiaByte` 
subroutine, a summary of the four calls are shown in the table.

|Line|R12|R13|Purpose|
|:-:|:-:|:-:|:-:|
|70|0x01|0xE7|Writes the 8 bit pattern to the LCD|
|280|0x00|0xBX|Sets the row address in the LCD|
|292|0x00|0x1X|Sets the "upper 3-bit" column address|
|298|0x00|0x0X|Sets the "lower 4-bit" column address|
X indicates the particular address bits that change from call to call, these
two bits are passed to the subroutine as R13.

Every time the button is pressed and the `#writeNokiaByte` subroutine is called,
the MOSI pin sends a SPI command to the LCD. The signals were read using a 
logic analizer. For this test probe 0 was attached to pin P1.0 to read the MOSI
output, probe 1 was attached to pin P1.5 to read the clock, and probe 3 was 
attached to pin P1.7 to read the MOSI output. The trigger was set to the 
falling edge of the "component select low" pinEach of these subroutine calls
and the data packet are summarized below.

|Line|Command/Data|8-bit packet|
|:-:|:-:|:-:|:-:|
|70 (159)*|Data|1110 0111|
|280(500)|Command|1011 0001|
|292(512)|Command|0001 0000|
|298 (518)|Command|0000 0001|

* Line numbers in parentheses are where the routines were called after the 
functionlaity code had been added

These four bites represet the first time the S3 button was pressed and produced 
an output. The first written byte was the data which wrote the broken bar 
design to the display. The next three command bytes wite the address. The first 
byte shown in the table the row address, incremented to 1. The second two bytes 
represent the upper and lower bits of the colunm address respectively which has 
also been incremented to 1 in the code. Below are the screenshots of the logic 
analizer output that produced these four codes.

Data Write
![alt text](https//raw.githubusercontent.com/IanGoodbody/ECE382_Lab3/master/logicOutput/GBdata.png)

Row Set
![alt text](https://raw.githubusercontent.com/IanGoodbody/ECE382_Lab3/master/logicOutput/GBcmd1.png)

ColumnSet
![alt text](https://raw.githubusercontent.com/IanGoodbody/ECE382_Lab3/master/logicOutput/GBcmd2.png)
![alt text](https://raw.githubusercontent.com/IanGoodbody/ECE382_Lab3/master/logicOutput/GBcmd2.png)

In order to read the reset data pin, probe 3 was attached to pin 2.0, the reset
signal, and the trigger was set to read at the falling edge of that reset 
signal. The reset was read by pressing the hardware reset button S1 on the 
MSP430. The waveform is displayed below:

![alt text](https://raw.githubusercontent.com/IanGoodbody/ECE382_Lab3/master/logicOutput/GBreset1.png)

The code writes `0` to the reset pin inside the `#initNokia` subroutine which
holds the low value to the pin for count of 0xFFFF or 65535. The logic 
analyzer readout of this section, on the scale shown above, shows a high 
"blip" in the reset signal, followed by a longer low signal, then followed 
by a similarly long high signal where the board is not reset. This pattern 
matches the code in `initNokia` which has consecutive reset low then high 
signals set to a loop. Measuring between the blip and the long high period 
gave a period of 19.77 ms, and given 65,535 cycles of the loop, yields about 0.3 us per loop of the code segment shown below.

```Assembly
	bic.b	#LCD1202_RESET_PIN, &P2OUT
	mov	#0xFFFF, R12
delayNokiaResetHigh:
	dec	R12
	jne	delayNokiaResetHigh
	; This loop creates a nice delay for the reset high pulse
	bis.b	#LCD1202_RESET_PIN, &P2OUT
```

#### Increased functionality

The B and functionalities required that the MSP430 write a 8X8 block to the LCD
screen for the B functionality and for that block to move around the screen
in response to button presses for the A functionality.

##### Write Modes

An alternate method for writing data to the LCD screen, as opposed to simply
overwring the entire screen (which would require storing more data bits in RAM 
than there is available), is to use certain "write modes" to read write only
the data that needs to be changed.

Example mappings of these write modes are shown below.

![alt text](https://raw.githubusercontent.com/IanGoodbody/ECE382_Lab3/master/bitblock.bmp)

Unfortunately, the designer was unable to find any write mode setting commands 
inherant in the LCD microcontroller codes. It would be possible to save 
the pertinant data in RAM and modify it using these write modes using the 
MSP430 commands. The `xor` option proves particularly attractive because the 
overwriting method would only require that the bits that need to be changed be 
written to the saved data.

##### B Functionality

B functionality was fairly easily achieved without adding any extra subroutines
by simply writing `0xFF` to eight times in a loop and incrementing the column
parameter with each iteration. To add some variety to the program, after each
block was drawn, the page was incremented then modded with 8 to create a series
of cascading blocks.

##### A Functionality

A functionality requeid that the block move about the LCD screen. The way that
this functionality was implemented allowed the designer to provide a variety of
different images (for the demonstration a smiley face was chosen), additionally
the designer decided that the image woudl move only 1 pixel per button press 
and that the image will not be allowed to move once it reaches the end of the
LCD screen.

Two new subroutines were added in order to implement this functionality. The
`drawOffCol` subroutine draws a passed byte to a pixel cursor address. This 
subroutine allows the designer to draw images over multiple pages by shifting
the image bits across two write registers and writing them to seperate pages.
`drawPattern` allows the user to draw a series of colums as a coherant image on
any pixel location on the screen. This subroitine is passed the pixel address
of the top left corner of the image, the memory address of the image byte array,
and the length of that byte array.

In implementing these two methods the `main:` loop of the program checks that
the addresses are within the valid range to not "fall off" of the LCD screen, 
as well as clearing the previous image before writing the moved image. Clearing
the image requied writing a full sequence of `0x00` bytes over the previous 
image as the overwrite process will not consistently clear old bits once the 
cursor moves to a new page. The left and right move clears could be easily
acomplished by clearing the extreme colums of the image with the `drawOffCol`
subroutine, and allowing the new image to overwrite the rest of the old. 

Because the design uses hard coded values for the cursor address ranges and the
message length, the current implementation will work best only with 8 by 8
images. Creating images of variable width could be acomplished by chanigg the
parameters noted above, however images with heights other than 8 are not 
supported by this implementation.
