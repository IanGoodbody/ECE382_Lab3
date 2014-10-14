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
the MOSI pin sends a SPI command to the LCD. These commands and their contens 
are sumarized below.

|Line|Command/Data|8-bit packet|
|

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

![AltText]("https://raw.githubusercontent.com/IanGoodbody/ECE382_Lab3/master/bitblock.bmp")

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
