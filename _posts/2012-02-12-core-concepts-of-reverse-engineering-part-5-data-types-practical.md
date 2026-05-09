---
tag: Reverse Engineering
---

# Core Concepts Of Reverse Engineering: Part 5 – Data Types Practical

In the last article I talked about some theory of how data types are stored in binary. Of particular importance were the concepts of endianness which defines the order of the bytes that make up a data type. This article will use XVI32 to practice determining and changing the values of data entries which make use of some of the data types we discussed in the last article.

## Determining Data Values

I’ve put together a small sample file for this article, before continuing you’ll need to download it and open it in XVI32.

[Download here](https://mvi.sh/permanent/coreconcepts/part5.sample)

<img src="/Images/ReverseEngineeringPart5Image1.png" />

Once you have the file open in XVI32 you should see the hex and text values in the above screenshot.

To make this article more straightforward I’m going to tell you the actual structure of this file:

<img src="/Images/ReverseEngineeringPart5Image2.png" />

Remember that there are 8 bits to a byte, so get the number of bytes in a 32 bit integer you divide 32 by 8 and get 4.

In this part of the practical, you need to answer four questions:

1. What is the decimal (base 10) value of the byte at offset 0?
1. What is the decimal (base 10) value of the little-endian int16 at offset 1?
1. What is the decimal (base 10) value of the big-endian int32 at offset 3?
1. What is the decimal (base 10) value of the little-endian int32 at offset 7?

### Tips for the struggling

If you struggle with question 1, reread part 2.

If you struggle with questions 2 or 4, reread part 4.

If you struggle with question 3, remember that the conversion is the same as for little-endian numbers only you don’t need to bother with reversing the order of the bytes.

### Answers

The answers can be found [here](https://mvi.sh/permanent/coreconcepts/part5answers.txt). If you didn’t get the same answer, check out the tips above.

## Changing Data Values

Before starting this section, you need to make sure you’ve completed the questions above and got the correct answers to every question.

In this section, we want to change the values of the data entries encoded in this sample file. If you want to refresh your memory of making hex edits with XVI32, reread part 3 now. Remember to use overwrite mode in XVI32 rather than insert.

Again we have four exercises:

1. Change the value of the byte to 54 (expressed as a decimal integer)
1. Change the value of the little-endian int16 to 40 (expressed as a decimal integer)
1. Change the value of the big-endian int32 to EDA0 (expressed as a hexadecimal)
1. Change the value of the little-endian int32 to 6767 (expressed as a decimal integer)

If you struggle with any of these questions, the best idea is to reread the previous articles.

### Answers

The answers can be found [here](https://mvi.sh/permanent/coreconcepts/part5-answers.sample). Open this file in XVI32 to compare against your own edits.

In this article you’ve determined the values of different data types and also changed those values. The skill you’ve just picked up means that you’re now able to hex edit a large range of data and files. Being able to determine the value of and edit data which occupies more than one byte is the most core practical skill in hex editing and reverse engineering binary files.
