---
tag: Reverse Engineering
---

# Core Concepts Of Reverse Engineering: Part 2 – The Hex Editor

Before I start this article, I need to define a couple of terms:

1. Byte – This is a data type which can store 256 discrete variations. Typically its said to have a minimum value of 0 and a maximum value of 255 (thus 256 variations including the 0). All files can be considered to just be a series of bytes.
1. Hex or Hexadecimal – Unlike a decimal or base 10 number which only allows 10 different variations per digit and must include a another digit to include numbers which exceed that range (e.g. 8, 9, 10, 11 / 98, 99, 100, 101) a hexadecimal number stores 16 variations. After 9, the first six letters of the alphabet are used (e.g. 8, 9, A, B, C, D, E, F, 10 , 11 / FE, FF, 100, 101) .

All files are divided into two categories – plain text and binary.

**Plain text** files as the name suggests just contains text. They cannot contain images, sound, video or any form of text styling unless they mark it up. Examples of these files include .txt, .ini, .csv, .html, .php. These files can be opened in your system’s default plain text editor such as Notepad on Windows and will load and display fine.

<figure>
<img src="/Images/ReverseEngineeringPart2Image1.png" />
<figcaption>Plain text files load and display fine in plain text editors</figcaption>
</figure>

**Binary files** however can store a much wider range of data. Your camera photos, mp3s and videos are all binary files. Rather than limit the data storage to just text, binary files can make use of a larger range of encoding which means that we can’t view these files properly in a plain text editor. This is shown in the image below, in which I’ve tried to load a bitmap image into Notepad.

<figure>
<img src="/Images/ReverseEngineeringPart2Image2.png" />
<figcaption>By contrast, loading a binary file in a plain text editor is not a good idea</figcaption>
</figure>

So to view and edit binary files, we clearly need a different tool. If we know the file format then we could load the file into the relevant editor, loading images into Photoshop or Paint for example. But this is no good to us if we don’t know the file format, if there isn’t a relevant editor yet or if we want to examine the internal data structure.

**Hex Editors** can display and edit binary data in a very helpful and effective way. They are not limited to text characters and can be used to display and edit the full range of variations in each byte. Unlike text editors which display binary data badly and don’t support changing the value of non text data, hex editors are not hindered by these problems.

<figure>
<img src="/Images/ReverseEngineeringPart2Image3.png" />
<figcaption>XVI32 is a freeware hex editor for Windows</figcaption>
</figure>

My personal favourite free hex editor is [XVI32](http://www.chmaas.handshake.de/delphi/freeware/xvi32/xvi32.htm) which can be downloaded for free on Windows. It’s quite a lightweight and functional hex editor and while there are many hex editors out there offering a greater range of features, I like the simplicity and straightforwardness of XVI32.

In the above screen shot we can see three columns. The left and thin column is the line number displayed in hex. The number shown represents the index or offset of the first entry on that line, for example B means 11 as a decimal and if you count across the boxes in either of the other two columns you’ll see that they are also 11 boxes across. These line groupings do not exist in the actual file, this is merely just how its displayed in the editor, a bit like word wrapping text.

The middle column displays the hexadecimal view of the file, while the right column shows a ASCII or text view of the file. Each box in the hex and text views represents a byte in the file.

## So why is hex useful? Why not represent the values of each byte as a decimal?

Hexadecimal numbers have the useful property that with two digits they can represent 256 discrete variations, just like a byte. So rather than use 3 digits to represent the value of each byte, we can use two digits to their full range. The minimum hex value for a byte is 0 (or 00) and the maximum hex value for a byte is FF.

To convert between decimal and hexadecimal you can use the built in calculator on Windows (or use a site like [this](http://www.statman.info/conversions/hexadecimal.html)). Depending on your version of Windows you may need to change to either scientific or programmer mode before the hex and decimal options are available. To convert a number, type it into one mode then select the other mode. For example:

<img src="/Images/ReverseEngineeringPart2Image4.png" />

<img src="/Images/ReverseEngineeringPart2Image5.png" />

In the next post, I’ll explore how to use a hex editor and look at some common data types. Before reading that post however, it would be useful to try opening a few different file types in a hex editor just to get a feel for it. It would also be very helpful to try converting a few different numbers between hex and dec.
