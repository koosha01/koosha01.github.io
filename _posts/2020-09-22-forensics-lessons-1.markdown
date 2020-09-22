---
layout: post
title:  "Forensics Lessons 1"
tags: forensics ctf
exerpt_seperator: <!--more-->
---
Recently I've been engaging with some forensics challenges. Honestly this was my first encounter with this type of challenges in the CTF style format.
  
I've discovered a lot of things during the progress of solving and learned a lesson or two.  
So I hope sharing them would be useful for you. Enjoy :)

## Network
<hr>  
In this category I'm talking about the case you're given a  pcap file to analyze the commutations between different sides present in the network.  
I used *tshark* as the tool for my analysis.

The first lesson I learned here is studying enough about the present protocols.  
A major time of mine was wasted because of the knowledge I was missing about some http request header.

So my advise here is to know everything needed for the protocol to be able to analyze the tshark/wireshark output completely.  
Don't dig too deep to get into things that are not related to the challenge, or too shallow not being able to understand what's going on.
    
The Wireshark display filter reference is so helpful for digging into the connections: https://www.wireshark.org/docs/dfref/

## File System
<hr>
- **Tools**

Tools play a big role in CTF.  
Here are the common tools which I used during the challenge:

1. strings: collecting human readable characters from the file

2. file: recognizing file format

3. binwalk: walking through a binary to find embedded files according to magic numbers

4. hexdump: dumping hex output of the file

5. python: as the scripting language for needed scripts 

6. xxd: converting a hexdump to binary and vice versa  


- **File Header**

Different files have different file signatures.  
Usually the first bytes of the file are considered as so. They're also called magic numbers.  
For example PNG files start with this sequence of hex values:  
	
	89 50 4E 47 0D 0A 1A 0A  

This way files can be identified as what they are.
  
A very significant point of view about files are these magic numbers. After using ordinary tools for collecting info, pay attention to the file's header and footer.  
Maybe it's corrupted and you should fix it this way.

- **Zlib**

Zlib is a data compression library used in various application software.  
During the progress of analyzing files I used to run the binwalk command on them, in several occasions I faced this message:

> Zlib compressed data

First I thought maybe it's a compressed file embedded in the main file. But I failed decompressing it using various tools around zlib compression library.  
At last I found out the main file was a PNG image. It made sense cause PNG uses Zlib as the means of compression.

So the lesson here is to look up for the software which uses a particular library you found in the main file using binwalk.

- **Zip and PNG Structures**

These are two important file types and the ones I've been working with in the challenge.  
A bit of knowledge about both would be nice.

1. Zip:
  
    A Zip file basically consists of two main parts:
    
    + Central Directory: Keeps track of all the files in the archive.
    + File Entries: Files in the archive, contain local file headers having info about the file.
    
    Central directory keeps records of the files inside the archive. It has different info such as file name, metadata, an offset which points to the actual data in the archive and etc.
    
    This way the listing of the archive would be done fast and easily. Central directory is also responsible for the updates which will be made on the archive, including deleting a file, adding a new one and so on. Meaning that scanning for local file headers through the archive would be invalid.
    
    A true Zip file identification is by the presence of *end of the central directory* which is displayed by these set of bit:  
    
        50 4b 05 06
    
    So no general BOF or EOF.
    
2. PNG:
	
	It starts with this 8-byte signature:
	   
		89 50 4e 47 0d 0a 1a 0a
		
	So after the header there comes a series of chunks. Chunks are divided into two sets, critical and ancillary.  
	A brief explanation of them is as so:
	
	1. IHDR: First chunk, contains basic information about the image like width, height, color type and ... .
	
	2. IDAT: Contains the actual image data coming from the compression algorithm.
	
	3. IEND: Marks the end of the file.         

<br/><br/>
So that was it! Hope you learned a thing or two ;)


      


