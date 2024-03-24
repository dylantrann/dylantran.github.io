---
title: "Huffman File Compressor"
excerpt: "File compressor and decompressor that uses a Huffman encoding<br/><img src='/images/huffmantree.png'>"
collection: projects
---

This was a project assigned as part of the course CSE 100R taught by Niema Moshiri at UCSD. It uses a Huffman encoding to compress files and contains a decompression algorithm to later decompress the file without having access to the original. It's coded **purely in C++** and makes use of **file I/O** and **advanced data structures**. 

Course code can be made available upon request, but not normally public due to course policies. 

# Usage
There are two main functions of this program, which are `compress` and `uncompress`. To use the program, you just have to follow these steps:

1. Run `/make` to compile the program using the given `Makefile`
2. To compress, use the command `./compress <original_file> <compressed_file>`, compressing the contents of `original_file` and outputting the results into `compressed_file`
3. To decompress, just use `./uncompress <compressed_file> <uncompressed_file>`, outputting the contents of the `original_file` into `uncompressed_file` 

The program was tested normally with binary and text files but, in practice, it should work with any file using ASCII's general 256 characters. 

# Project Overview
I created and coded three files for this project: `HCTree.cpp`, `compress.cpp`, and `uncompress.cpp`. 

## 1. `HCTree.cpp`
This file contains the foundational Huffman tree data structure for encoding and decoding purposes. 

For the `HCTree::build` function, I used a min-heap to efficiently manage the construction of the Huffman tree. Since C++'s default priority queue is in the form of a max-heap, I had to specify a comparison overloader, `HCNodePtrComp`, which was provided to us in the write-up. During the creation of the Huffman tree, I saved the order in which unique symbols were first added into the Huffman tree to later use that information for the compressed file header. The compressed file header is a big part of both `compress.cpp` and `uncompress.cpp` and will be discussed in further detail in their sections.

`HCTree::encode` takes the symbol to be encoded, given in its parameters, and finds it's associated leaf node. It then backtracks its path from leaf to root in the Huffman tree, appending each bit in it's Huffman encoding in the form of a string. Once the encoding is complete, it is then outputted into the `<compressed_file>` using `Helper.cpp`'s FancyOutputStream object. 

`HCTree::decode` reads bits from input, which is the `<compressed_file>`, and uses those bits to find the symbol encoded. It starts at the root of the tree and chooses the child to traverse down to by the given bit from input. Once a leaf node is reached, it's given symbol is returned. 

The destructor method `HCTree::~HCTree` just uses a simple recursive function with post-order traversal to delete each node of the Huffman tree.

## 2. `compress.cpp` 
This program reads data from a starting file, `<original_file>`, and outputs a Huffman encoded version of it to an output file, `<compressed_file>`. 

It starts by reading through `<original_file>` and tracking the number of times each unique symbol, represented by a byte of information, appears. Using that information stored in a vector, we construct our Huffman tree using `HCTree.cpp`. 

Then we begin encoding the **compressed file header**. The compressed file header is used to store information to construct the given Huffman tree, which is necessary because the `uncompress` program lacks access to the original file, meaning it cannot reconstruct the Huffman tree without extra encoded data. 

To create the file header, my idea was to output each symbol along with their frequency consecutively and mark the end of the header (i.e. the beginning of the actual encoding) using the null character, `\0`. The main issue to was figuring out how to efficiently encode the frequency of a symbol, since a symbol can appear over 255 times, meaing it can exceed max number that can be represented by a byte of information. Other than a byte, we could only encode each symbol as an integer (4 bytes) but that would be extremely inefficient regarding a case in which there was 1 of each symbol, each with a frequency of also 1. 

Thus, I decided on a straightforward solution. I would track the symbols with frequencies less than 256 separate from symbols with frequencies above that threshold, solving the issue without wasting too much space nor brainpower. Thus, I used the global variable `order` present in an HCTree object to output symbols and their frequencies in order from least to greatest. This is done in bytes until a symbol with a frequency greater than 255 is read, in which we pause and insert a null character before continuing using bytes for symbols and integers for their following frequencies. Once we've printing all symbols and their frequences, we end with another null character and begin outputting our encoding. For easier reading with the null character markers, I printed each symbol's frequency before the symbol themselves. 

For example, the file header of a Huffman tree created with 5 of the symbol 1 and 300 of the symbol 2 would look like:

```
0101 0001 0000 0000000100101100 0010 0000 # spaces not normally present, only for easier reading
# 5 1 0 300 2 0
```

From there we just reset our FancyInputStream object and read through the entire `<original_file>` using `HCT::encode` to encode and output each character we encounter. 

## 3. `uncompress.cpp`
This program reads from a compressed file, `<compressed_file>`, decodes it, and outputs its original contents into another file, `<uncompressed_file>`.

It starts by decoding the compressed file header, which will provide information required to reconstruct the Huffman tree for decoding purposes. It reads the symbols and frequencies from input in bytes until reaching a null character. It will then alternate between reading in integers and bytes until reaching another null character, in which we pause our file I/O to construct our Huffman tree. It should be mentioned that each time we encounter a frequency reading, we sum that with the int `totalCount`, which keeps track of the total characters in the original file. 

Once the Huffman tree is constructed, we can begin bitwise decoding using `HCTree:decode` and outputting the given character into the `<uncompressed_file>`. We continue decoding this way until `totalCount` number of characters has been outputted, in which the remaining bits are just filler 0's flushed from the buffer and must be ignored. Thus, our uncompression is complete and our outputted file should be identical to our original. 

