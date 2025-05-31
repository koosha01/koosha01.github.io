---
layout: post
title:  "Beginner Crypto Toolkit"
tags: cryptography ctf python encoding
excerpt_separator: <!--more-->
---
When it comes to CTF challenges and cybersecurity, having a proper toolkit becomes an essential part; enabling one to use the underlying knowledge with ease. Personally I rather centralize my practice of cryptography in Python.  
<!--more-->

An inseparable part of cryptography in the internet environment is encoding; Integer, hexadecimal, binary, Ascii and so on.  
In this post I'm gonna provide a set of commands and a few tricks, to convert one encoding to another and perform the desired transformations, resulting in a handy platform to deal with in practice cryptography. Enjoy :)  

## Basic Encodings
---

In each section we'll see the commands converting other encodings to the entitled one.

### Integer

- Numbers

To Convert a number to an integer.

|Command|Format|Parameters|Return Type|
|:---:|:---:|:---:|:---:|
|int|int(x, base)|x: str <br> base: int|int|

Examples:
```
>> int("0a", 16)
Output: 10
```
```
>> int("110", 2)
Output: 6
```
```
>> int("123")
Output: 123
```

- Character

To return the Unicode code point for a one-character string.

|Command|Format|Parameters|Return Type|
|:---:|:---:|:---:|:---:|
|ord|ord(c)|c: one-character str|int|

Examples:

```
>> ord("A")
Output: 65
>> ord("\n")
Output: 10
```

- Bytes

Using the bytes object as a list, we can access each byte as an integer value.

Examples:

```
>> d = b'\x90\x90Hi'
>> print(d[0])
Output: 144
>> print(d[1])
Output: 144
>> print(d[2])
Output: 72
>> print(d[3])
Output: 105
```

### Hex

- Integer

To return the hexadecimal representation of an integer.

|Command|Format|Parameters|Return Type|
|:---:|:---:|:---:|:---:|
|hex|hex(x)|x: int|str|

Examples:

```
>> hex(100)
Output: '0x64'
```

### Bytes

- Hex

To create a bytes object from a string of hexadecimal numbers.

|Command|Format|Parameters|Return Type|
|:---:|:---:|:---:|:---:|
|bytes.fromhex|bytes.fromhex(s)|s: str |bytes object|

Examples:
```
>> bytes.fromhex("0a")
Output: b'\n'
```
```
>> bytes.fromhex("41 0a")
Output: b'A\n'
```

- Integer

To convert an iterable of ints to bytes.

|Command|Format|Parameters|Return Type|
|:---:|:---:|:---:|:---:|
|bytes|bytes(x)|x: an iterable of ints |bytes object|

Examples: 

```
>> i = [72, 101, 108, 108, 111]
>> bytes(i)
Output: b'Hello'
```

### Ascii

- Bytes

To decode the bytes using the codec registered for encoding.

|Command|Format|Parameters|Return Type|
|:---:|:---:|:---:|:---:|
|bytes.decode|bytes.decode(self, encoding)|encoding: str |str|

Examples:

```
>> d = bytes.fromhex("41")
>> d.decode("ascii")
Output: 'A'
>> d.decode("utf-8")
Output: 'A'
>> d.decode()
Output: 'A'
```

- Integer

To return a Unicode string of one character.

|Command|Format|Parameters|Return Type|
|:---:|:---:|:---:|:---:|
|chr|chr(i)|i: int |str|

Examples:

```
>> chr(110)
Output: 'n'
>> chr(123)
Output: '{'
```


## One for All
---

There is a library in python named *codecs*. We can use the *decode* and *encode* functions to perform various text and binary transformations.


|Command|Format|Parameters|Return Type|
|:---:|:---:|:---:|:---:|
|decode|decode(obj, encoding)|obj: depending on the encoding <br>encoding: str |depending on the encoding|

> The command format of *encode* is similar.

List of useful encodings:

|Codec|Meaning|
|:---:|---|
|ascii|Convert the operand to ascii.|
|base64|Convert the operand to multiline MIME base64 (the result always includes a trailing '\n').|
|bz2|Compress the operand using bz2.|
|hex|Convert the operand to hexadecimal representation, with two digits per byte.|
|zip|Compress the operand using gzip.|
|rot13|Return the Caesar-cypher encryption of the operand.|

> You can see the full list [here](https://docs.python.org/3/library/codecs.html).

Examples:

```
>> from codecs import encode
>> d = b'Hello'
>> encode(d, "base64")
Output: b'SGVsbG8=\n'
>> encode(d, "zip")
Output: b'x\x9c\xf3H\xcd\xc9\xc9\x07\x00\x05\x8c\x01\xf5'
>> encode(d, "hex")
Output: b'48656c6c6f'
```

```
>> from codecs import decode
>> d = b'Hello'
>> decode(d, "ascii")
Output: 'Hello'
>> decode(b'48656c6c6f', "hex")
Output: b'Hello'
```

## A few Tricks
---

### zip

Using the *zip* python command we can construct tuples out of our inputs, suitable to perform binary operation upon them.

|Command|Format|Parameters|Return Type|
|:---:|:---:|:---:|:---:|
|zip|zip(iterables)| iterables |A zip object yielding tuples until an input is exhausted.|

Examples:

```
>> list(zip('abcdefg', range(3), range(4)))
Output: [('a', 0, 0), ('b', 1, 1), ('c', 2, 2)]
>> list(zip('AAA', 'BBBB'))
Output: [('A', 'B'), ('A', 'B'), ('A', 'B')]
>> list(zip(b'AAA', b'BBBB'))
Output: [(65, 66), (65, 66), (65, 66)]
```

Application:

Using zip to implement a byte-by-byte xor to a ciphertext with a key.

```
>> key = b'happy'
>> cipher =  b'hellothereiamheretotestthiscommand'
>> bytes([c ^ k for c, k in zip(cipher, key)])
Output: b'\x00\x04\x1c\x1c\x16'
```

## Resources 
---

- Python Documents