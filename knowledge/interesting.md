# Binary Format
Binary formats are often used for data that needs to be transmitted over a network or stored in a file, as they can be more compact and efficient than text-based formats.

In the case of `HTTP/2`, the binary format is used to serialize the data that is being transmitted between the client and the server. This means that the data is represented as a sequence of binary values, which can be more efficient to transmit than a text-based format such as the one used in `HTTP/1.x`.

The same block of computer memory can be interpreted in three different ways:
```
First, we treat each byte as a single character.

0  :  74 65 73 74  |  test

Next, we interpret the memory as two, two-byte integers.

0  :  74 65 73 74  |  25972 29811

Finally, we can also interpret the memory as a single, four-byte real number.

0  :  74 65 73 74  |  7.713537e+31
```

One of the binary format to store file is called network Common Data Form (netCDF).

Like most binary formats, the structure of a netCDF file consists of header information, followed by the raw data itself. The header information includes information about 
- how many data values have been stored
- what format the values have been stored in
- where within the file the header ends and the data values begin

For example, we have a file contains many temperature values. Suppose the raw data is located at **byte 624** within the file and each temperature value is stored as an **eight-byte real number**.

```
Header information:
 0  :  43 44 46 01 00 00 00 00 00 00 00 0a  |  CDF.........
12  :  00 00 00 01 00 00 00 04 54 69 6d 65  |  ........Time
24  :  00 00 00 30 00 00 00 00 00 00 00 00  |  ...0........
36  :  00 00 00 0b 00 00 00 02 00 00 00 04  |  ............
48  :  54 69 6d 65 00 00 00 01 00 00 00 00  |  Time........

Raw data:
624  :  40 71 6e 66 66 66 66 66  |  278.9
632  :  40 71 80 00 00 00 00 00  |  280.0
640  :  40 71 6e 66 66 66 66 66  |  278.9
648  :  40 71 6e 66 66 66 66 66  |  278.9
656  :  40 71 5c cc cc cc cc cd  |  277.8
664  :  40 71 41 99 99 99 99 9a  |  276.1
672  :  40 71 41 99 99 99 99 9a  |  276.1
680  :  40 71 39 99 99 99 99 9a  |  275.6

Plain text data for contrast:
359  :  32 37 38 2e 39  |  278.9
```
Binary format advantage:
- Data can be accessed quickly than plain text format
  - All data values have same length. It is possible to calculate the location of a data value within the file. For example, if we want to access the fifth temperature value, 277.8, within this file, we know with certainty, because the header information has told us, that this value is 8 bytes long and it starts at byte number 656: the offset of 624, plus 4 times 8 bytes (the size of each temperature value).
  - In general, it is possible to jump directly to a specific location within a binary format file, whereas it is necessary to read a text-based format from the beginning and one character at a time. This feature of accessing binary formats is called **random access** and it is generally faster than the typical **sequential access** of text files.
- Can use up less memory than plain text format in general

Binary format disadvantage
- May use up more memory than plain text format in some cases
  - We can obviously tell that to store the temperature value in plain text, only 5 bytes are needed, which are smaller than 8 bytes.
- Require understanding of the data format (eg. need metadata), harder to share data and may even need a specific software to work with binary format.


Reference: https://www.stat.auckland.ac.nz/~paul/ItDT/HTML/node39.html
