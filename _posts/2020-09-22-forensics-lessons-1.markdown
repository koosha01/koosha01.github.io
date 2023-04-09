---
layout: post
title:  "Forensics Lessons"
tags: forensics ctf
excerpt_separator: <!--more-->
---
Recently I've been engaging with some forensics challenges. Honestly this was my first encounter with this category in [CTF](https://en.wikipedia.org/wiki/Wargame_(hacking)).
  
I've discovered a lot during the progress of solving and learned a lesson or two.  
So I hope sharing them would be useful for you as well. Enjoy :)
<!--more-->

## Network
--- 
By "Network" I mean the case in which you're given a pcap file to analyze the commutations between different parties present in a network.  
I used *tshark* as the tool for my analysis.

The first lesson I learned here was to study enough about the **present protocols**.  
In my case there were two sides, one sending http packets to another, transferring a *PNG* file. The catch is, the order was wrong. So my job was to sort the packets in the right order, and then putting them together.  
A major time of mine wasted because of the knowledge I was missing about some http request header, so I couldn't sort the packets properly and got stuck for a good deal of time.

My advise here is to absorb the necessary knowledge about the protocol, so you can analyze the tshark/wireshark output well.  
Don't dig too deep, getting into things not related to the challenge, or too shallow not being able to understand what's going on.
    
The [Wireshark display filter reference](https://www.wireshark.org/docs/dfref/) is so helpful to set the tshark/wireshark parameters right and get a nice analysis done.


## File System
---
### Tools

Tools play a big role in CTF.  
Here are some common tools I used during the challenge:

1. strings: Collects human readable characters from the file.

2. file: Recognizes the file format.

3. binwalk: Walks through a binary to find embedded files according to magic numbers.

4. hexdump: Dumps hex output of the file.

5. python: As the magical scripting language! 

6. xxd: Converts a hexdump to binary and vice versa.

Note: Python is a great help in CTFs. A good scripting language is always needed, empowering you to do all kinds of computations and inspections, from deciphering to packet analyzing. 


### File Header

Different files have different file signatures. This way they can be identified as what they are.  
Usually the first bytes of the file are considered as such. They're also called magic numbers.  
For example PNG files start with this sequence of hex values:  
	
	89 50 4E 47 0D 0A 1A 0A  
  
A very significant point of view about files are these magic numbers. After using ordinary tools for collecting info, pay attention to the file's header and footer.  
Maybe it's corrupted and you should fix it this way. In some cases (including mine!) they miss a few characters among the sequence.

### Zlib

Zlib is a data compression library used in various application software.  
During the progress of analyzing binaries I used to run the binwalk command on them, in several occasions I faced this message:

> Zlib compressed data

First I thought maybe it's a compressed file embedded in the main binary. But I failed decompressing it using various tools around zlib compression library.  
At last I found out the binary was a PNG image, using Zlib library as the means of compression. It makes sense cause PNG has a compression process among other tasks.

So the lesson here is when you execute binwalk and find a library, look up for various software which make use of that library. In our case the PNG file format which includes Zlib for data compression.


### Zip and PNG Structures

These are two important file types and the ones I've been working with in the challenge.  
A bit of knowledge about both would be nice.

- **Zip:**
  
    A Zip file basically consists of two main parts:
    
    + Central Directory: Keeps track of all the files in the archive.
    + File Entries: Files in the archive, contain local file headers having info about the files.
    
    Central directory keeps records of the files inside the archive. It has different info such as file name, metadata, an offset which points to the actual data in the archive and etc.
    
    This way the listing of the archive would be done fast and easily. Central directory is also responsible for the updates which will be made on the archive, including deleting a file, adding a new one and so on. So scanning for local file headers through the archive would be invalid act.
    
    A true Zip file identification is done by presence of *end of the central directory* which is displayed by these set of bits:  
    
        50 4b 05 06
    
    So no general BOF nor EOF.
    
- **PNG:**
	
	It starts with an 8-byte signature:
	   
		89 50 4e 47 0d 0a 1a 0a
		
	After the header there comes a series of chunks.  
	What is a chunk you may ask? It's simply a part (chunk) of the file representing a bunch of info. Chunks are divided into two sets, critical and ancillary.  
	Brief explanation of the main chunks:
	
	1. IHDR: First chunk, contains basic information about the image like width, height, color type and ... .
	
	2. IDAT: Contains the actual image data coming from the compression algorithm.
	
	3. IEND: Marks the end of the file.
	
	Some ancillary chunks:
	
	1. 	tIME: Stores the last time the file was changed.
	2. 	tEXt: Can store text.
	<br>.<br>.<br>.
	         

	Depending on the image there could be much more, you can look them up [online](https://en.wikipedia.org/wiki/Portable_Network_Graphics#%22Chunks%22_within_the_file).
	
	Some basic tools regarding PNG format file:
	1. 	pngtools: It's an Ubuntu package which includes a series of tools for inspecting PNG images, such as: pnginfo, pngchunks and ... .
	2. 	pngcheck: Checks for file integrity and displays some info.
	3. 	pngmeta: Extracts meta data out of the image.

	If the task is about extracting some hidden information out of the image, there are much more advanced tools you can find online which lie down in [steganography](https://en.wikipedia.org/wiki/Steganography) category.

<br/><br/>
So that was it! Hope you learned a piece ;)


      


