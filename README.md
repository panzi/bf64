bf64
====

Unpack, list and mount BF64 archives. These archives can be found in certain
Ubisoft games.

Basic usage:

	bf64.py list <archive>                 - list contens of .bf archive
	bf64.py unpack <archive>               - extract .bf archive
	bf64.py mount <archive> <mount-point>  - mount archive as read-only file system

The `mount` command depends on the [llfuse](https://code.google.com/p/python-llfuse/)
Python package. If it's not available the rest is still working.

This script is compatible with Python 2.7 and 3.

File Format
-----------

There is a lot of guesswork here. It is possible that I misaligned the
file entries and I know there is something more in the header of the archive.

Byte order is little endian and the character encoding seems to be 7-bit
ASCII. All offsets are absolute from the start of the file.

Basic layout:

	File header
	Chunk
		Chunk header
		File info table
		File data table
	Chunk...

### File Header

Size: 752 (at least in one example I saw)

There seems to be a table structure with several entries, but most are
empty. Two entries contain the strings `"ROOT"` and `"Bin"`. I don't have
enough data to find out more about this.

	Offset  Size  Type          Description
	     0     4  char[4]       file magic "BF64"
	     4   748  ?             ? (possible table of variable size)

### Chunk Header

Size: 12

	Offset  Size  Type          Description
	     0     4  int32_t       number of files
	     4     4  int32_t       next chunk offset
	     8     4  int32_t       ?

### File Info

Size: 137

	Offset  Size  Type          Description
	     0     4  ?             ?
	     4     4  int32_t       file data offset (see next section)
	     8     4  int32_t       always 0
	    12     4  int32_t       file size
	    16     4  int32_t       1-based file index, except for file 4, where it's -1
	    20     4  int32_t       previous field - 2, except for file 4 (2) and file 5 (-1)
	    24     4  int32_t       usually 1, for 4 files it's 0
	    28     3  ?             ?
	    31     1  uint8_t       always 72
	    32    64  char[64]      zero padded filename
	    34    73  ?             ?

### File Data

The file data offset in the file info structure points to the file size field
of the file data structure. So you need to add 4 to that offset to get the
location of the actual file data.

	Offset  Size  Type          Description
	     0     4  int32_t       file size (N)
	     4     N  uint8_t[N]    file data

Related Projects
----------------

* [bgebf](https://github.com/panzi/bgebf): unpack, list and mount Beyond Good and Evil .bf archives
* [fezpak](https://github.com/panzi/fezpak): pack, unpack, list and mount FEZ .pak archives
* [psypkg](https://github.com/panzi/psypkg): pack, unpack, list and mount Psychonauts .pkg archives
* [unvpk](https://bitbucket.org/panzi/unvpk): extract, list, check and mount Valve .vpk archives
* [u4pak](https://github.com/panzi/u4pak): unpack, list and mount Unreal Engine 4 .pak archives
* [t2fbq](https://github.com/panzi/t2fbq): unpack, list and mount Trine 2 .fbq archives

BSD License
-----------

Copyright (c) 2018 Mathias Panzenböck

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.