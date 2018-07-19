BO (Binary Output)
==================

The Swiss army knife of data examination and manipulation.

This is the tool to use when you need to visualize or convert data in different formats.

Bo reads data and interprets it according to its current input mode, converting it to a binary format. It then re-interprets the binary data according to the output mode and writes it to output. This allows you to do all sorts of interesting things with your data:

**Note for examples**: i(whatever) sets input type, o(whatever) sets output type, P(whatever) sets preset to string (s) or C-style (c). Everything else is data. See section 'Commands' for a full explanation.


#### See the per-byte layout of a larger integer in big or little endian format.

    $ bo -n oh1 Ps ih4b 12345678
    12 34 56 78

    $ bo -n oh1 Ps ih4l 12345678
    78 56 34 12


#### Convert integers between bases.

    $ bo -n oh8b16 Pc ii8b 1000000
    0x00000000000f4240

    $ bo -n oi8b ih8b 7fffffffffffffff
    9223372036854775807

    $ bo -n ob8b ih8b ffe846134453a8c2
    1111111111101000010001100001001101000100010100111010100011000010


#### Examine part of a memory dump (faked in this case) as an array of 32-bit big-endian floats at precision 3.

Fake a 12-byte memory dump as an example:

    $ bo oB1 if4b 1.1 8.5 305.125 >memory_dump.bin

What it looks like in memory:

    $ bo -n -i memory_dump.bin oh1b2 Pc iB1
    0x3f, 0x8c, 0xcc, 0xcd, 0x41, 0x08, 0x00, 0x00, 0x43, 0x98, 0x90, 0x00

What it looks like interpreted as floats:

    $ bo -n -i memory_dump.bin of4b3 Pc iB1
    1.100, 8.500, 305.125


#### Convert the endianness of a data dump.

    $ bo oB1 ih2b 0123 4567 89ab >data.bin

    $ bo -n -i data.bin oh2l Pc iB1
    0x2301, 0x6745, 0xab89


#### Convert text files to C-friendly strings.

example.txt:

    This is a test.
    There are tabs  between these words.
    ¡8-Ⅎ⊥∩ sʇɹoddns osʃɐ ʇI

    $ bo -n -i example.txt os iB1
    This is a test.\nThere are tabs\tbetween\tthese\twords.\n¡8-Ⅎ⊥∩ sʇɹoddns osʃɐ ʇI


#### Reinterpret one type as another at the binary level (for whatever reason).

    $ bo -n oi4b if4b 1.5
    1069547520


#### Build up test data for low level code.

    $ bo -n oh1l2 Pc if4b 1.5 1.25 ii2b 1000 2000 3000 ih1 ff fe 7a ib1 10001011
    0x3f, 0xc0, 0x00, 0x00, 0x3f, 0xa0, 0x00, 0x00, 0x03, 0xe8, 0x07, 0xd0, 0x0b, 0xb8, 0xff, 0xfe, 0x7a, 0x8b



Building
--------

Requirements:

  * CMake
  * C/C++ compiler

Commands:

    mkdir build
    cd build
    cmake ..
    make
    ./bo_app/bo -h



Tests
-----

    ./libbo/test/libbo_test



Usage
-----

    bo [options] command [command] ...


### Options

  * -i [filename]: Read commands/data from a file.
  * -o [filename]: Write output to a file.
  * -n Write a newline after processing is complete.
  * -h Print help and exit.
  * -v Print version and exit.


Specifying "-" as the filename will cause bo to use stdin (when used with -i) or stdout (when used with -o).
By default, bo won't read from any files, and will write output to stdout.

Commands may be passed in as command line arguments, and as files via the -i switch. You may specify as many -i switches as you like. Bo will first parse all command line arguments, and then all files specified via -i switches.



Commands
--------

Bo parses whitespace separated strings and interprets them as commands. The following commands are supported:

  * i: Set input type.
  * o: Set output type.
  * p: Set the prefix to prepend to every output value.
  * s: Set the suffix to append to every value except the last.
  * P: Set the prefix and suffix from a preset.
  * "...": Read a string value.
  * (numeric): Read a numeric value.

Bo reads a series of whitespace separated commands and data, and uses them to interpret input data and convert it for output. The commands are as follows:

  * i{type}{data width}{endianness}: Specify how to interpret input data
  * o{type}{data width}{endianness}[print width]: Specify how to re-interpret data and how to print it
  * p{string}: Specify a prefix to prepend to each datum output
  * s{string}: Specify a suffix to append to each datum output (except for the last object)
  * P{type}: Specify a preset for prefix and suffix.


Bo will attempt to interpret any non-command as data (number for anything numeric, and string for anything between quotes).

Data is interpreted according to the input format, stored in an intermediary buffer as binary data, and then later re-interpreted and printed according to the output format.

Strings are copied as bytes to the intermediary binary buffer, to be reinterpreted later by the output format.

For example, data input as hex encoded little endian 16-bit integers (ih2l), and then re-interpreted as hex encoded little endian 32-bit integers (oh4l8) with a space (s\" \") suffix would look like:

    input:  1234 5678 9abc 5432
    output: 78563412 3254bc9a

Before processing any data, both input and output types must be specified via `input type` and `output type` commands. You may later change the input or output types again via commands, even after interpreting data. Any time the output type is changed, all buffers are flushed.



### Input Type Command

The input specifier command consists of 3 parts:

  * Type: How to interpret incoming data (integer, float, decimal, binary, etc)
  * Data Width: The width of each datum in bytes (1, 2, 4, 8, 16)
  * Endianness: What endianness to use when storing data (little or big endian)


### Output Type Command

The output specifier command consists of 4 parts:

  * Type: How to format the data for printing (integer, float, decimal, string, etc)
  * Data Width: The width of each datum in bytes (1, 2, 4, 8, 16)
  * Endianness: What endianness to use when presenting data (little or big endian)
  * Print Width: Minimum number of digits to use when printing numeric values (optional).

#### Type

Determines the type to be used for interpreting incoming data, or presenting outgoing data.

  * Integer (i): Integer in base 10
  * Hexadecimal (h): Integer in base 16
  * Octal (o): Integer in base 8
  * Boolean (b): Integer in base 2
  * Float (f): IEEE 754 binary floating point
  * Decimal (d): IEEE 754 binary decimal (TODO)
  * String (s): String, with c-style encoding for escaped chars (tab, newline, hex, etc).
  * Binary (B): Data is interpreted or output using its binary representation rather than text.

##### Notes on the boolean type

As the boolean type operates on sub-byte values, its behavior may seem surprising at first. In little endian systems, not only does byte 0 occur first in a word, but so does bit 0. Bo honors this bit ordering when printing. For example:

    $ bo ob2b ih2b 6
    0000000000000110
    $ bo ob2l ih2l 6
    0110000000000000

Note, however, that when inputting textual representations of boolean values, they are ALWAYS read as big endian (same as with all other types), and then stored according to the input endianness:

    $ bo ob2b ib2b 1011
    0000000000001011
    $ bo ob2l ib2l 1011
    1101000000000000
    $ bo ob2b ib2l 1011
    0000101100000000
    $ bo ob2l ib2b 1011
    0000000011010000

#### Data Width

Determines how wide of a data field to store the data in:

  * 1 bytes (8-bit)
  * 2 bytes (16-bit)
  * 4 bytes (32-bit)
  * 8 bytes (64-bit)
  * 16 bytes (128-bit)

#### Endianness

Determines in which order bits and bytes are encoded:

  * Little Endian (l): Lowest bit and byte comes first
  * Big Endian (b): Highest bit and byte comes first

The endianness field is optional when data width is 1, EXCEPT for output type boolean.

#### Print Width

Specifies the minimum number of digits to print when outputting numeric values. This is an optional field, and defaults to 1.

For integer types, zeroes are prepended until the printed value has the specified number of digits.
For floating point types, zeroes are appended until the fractional portion has the specified number of digits. The whole number portion is not used and has no effect in this calculation.

#### Examples

  * `ih2l`: Input type hexadecimal encoded integer, 2 bytes per value, little endian
  * `io4b`: Input type octal encoded integer, 4 bytes per value, big endian
  * `if4l`: Input type floating point, 4 bytes per value, little endian
  * `oi4l`: Output as 4-byte integers in base 10 with default minimum 1 digit (i.e. no zero padding)
  * `of8l10`: Interpret data as 8-byte floats and output with 10 digits after the decimal point.


### Prefix Specifier Command

Defines what to prefix each output entry with.

Note: Examples have escaped quotes since that's usually what you'll be doing with command line arguments.

  * `p\"0x\"`: Prefix all values with "0x".
  * `p\"The next value is: \"`: Prefix all values with "The next value is: "


### Suffix Specifier Command

Defines what to suffix each output entry with (except for the last entry).

Note: Examples have escaped quotes since that's usually what you'll be doing with command line arguments.

  * `s\", \"`: Suffix all values with ", ".
  * `s\" | \"`: Suffix all values with " | "


### Preset Command

Presets define preset prefixes and suffixes for common tasks. Currently, the following are supported:

  * c: C-style prefix/suffix: ", " suffix, and prefix based on output type: 0x for hex, 0 for octal, and nothing for everything else.
  * s: Space separator between entries.



Usage Examples
--------------

Read binary file.bin and print out in "C" format as bytes:

    bo -i file.bin oh1l2 Pc iB

  * Set output type to hex, 1 byte per value, little endian (doesn't matter here), 2 digits per value.
  * Use the "c" preset, which sets prefix to "0x" and suffix to ", ".
  * Set input to binary. No more commands will be accepted. All input after parsing this argument will be treated as binary.
  * Read binary data from file1.bin and output as 0x01, 0xfe, 0x5a, etc.


Read 3 numbers as big endian 32-bit floats and store them as binary to file.bin:

    bo -o file.bin oB1 if4b "1.1 -400000 89.45005"

  * Set output type to binary.
  * Set input type to floating point, 4 bytes per value, big endian.
  * Set the output file to "file.bin"
  * Write three 32-bit ieee754 floating point values in binary to file.bin in big endian order.


Read a number as a little endian 64-bit floating point number and print out its byte values:

    bo oh1l2 if8l "s\" \"" 1.5

  * Set output type to hex bytes, 2 digit width.
  * Set input type to 64-bit little endian float.
  * Set the separator value to a space.
  * Interpret the number 1.5


Do a complex operation with command arguments and files:

    bo -i file1 -i file2 oh2b4 ih2l 1234 "5678 9abc"

  * set output type to hex, 2 bytes per value, big endian, minimum 4 characters per value.
  * set input type to hex, 2 bytes per value, little endian.
  * process "1234" as 16-bit hex.
  * process "5678 9abc" as 16-bit hex values.
  * process file1 as commands
  * process file2 as commands


Convert the endianness of a binary file containing 16-bit values:

    bo -i input.bin -o output.bin oB2l iB2b

  * Set output type to binary, 2 bytes per value, little endian.
  * Set input type to binary, 2 bytes per value, big endian.
  * Read binary data from input.bin
  * Write binary data to output.bin


Convert the endianness of a binary file containing 16-bit values, using stdin and stdout:

    cat somefile.bin | bo -i - oB2l iB2b > swapped.bin

  * Set output type to binary, 2 bytes per value, little endian.
  * Set input type to binary, 2 bytes per value, big endian.
  * Read binary data from stdin
  * Write binary data to stdout

Convert the string "Testing" to its hex representation using "space" preset:

    bo oh1b2 Ps ih1 "\"Testing\""

output: 54 65 73 74 69 6e 67



Libbo
-----

All of bo's functionality is in the library libbo. The API is small (4 calls, 2 callbacks) and pretty straightforward since all commands and configurations are done through the parsed data. The basic process is:

  * Build a context object according to your needs.
  * Call one or more process functions.
  * Flush and destroy the context.
  * Gather output and errors via the callbacks.

`test_helpers.cpp` shows how to parse strings, and `main.c` from bo_app shows how to use file streams.



Issues
------

  * IEEE754 decimal types are not yet implemented.
  * 128 bit values are not yet implemented.
  * 16-bit ieee754 floating point is not yet implemented.



License
-------

Copyright 2018 Karl Stenerud

Released under MIT license:

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
