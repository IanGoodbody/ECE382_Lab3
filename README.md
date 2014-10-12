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
