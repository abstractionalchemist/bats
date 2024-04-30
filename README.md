# Bats (binary abstract tree structure)

## Introduction

After about 20 years of industry experience, I've seen a lot of patterns and
cycles and waves. It's been amazing to watch the tremendous progress in some
areas, and what feels like an eternal spring of "sideways" innovation in
others - stuff that picks up the ball, moves around the field to find an
opening, but just winds up somewhere behind and to the left of where it started.
I'm not good at sports, but I think that made sense. It sometimes feels like a
lot of energy spent without getting any closer to the goal. If you've ever tried
to follow along with hype-driven development

so I'm trying to develop a new way of developing applications - a new
architecture built on top of a new foundation. I've been revisiting the history
of computer innovations and noticing which things have advanced and which have
been left behind. Looking at what bits of insight have been lost. For example,
Alan Kay speaks often about how his conception of object-oriented programming
has been misunderstood. He and many other luminaries believe our code bases are
bloated and we're building software all wrong. After being in the industry and
innovating in software myself, I feel that I have to agree. Like Alan Kay, I am
inspired by trying to understand computing at the scale of life and cells.
Thinking more ecologically instead of mechanically. Thinking at the systems and
"systems of systems" level instead of the imperative python program level.

I've thought a lot about these concepts and I've been workshopping ideas for a
new foundation. I'd like to share them with you to help me organize the ideas
and start working.

## Bits and Wits

The first foundational concept is a new way of thinking about structured binary
data. Claude Shannon developed information theory back in 1937, and using the
term "bit" to represent a single yes/no piece of information. His thoughts on
encoding/decoding and sending information has been foundational for computer
science and networking. He was even interested in the connection to DNA, which
uses 4 nucleotides (base 4 instead of binary) to encode amino acid chains.

I spent some time thinking about the potential value of using a base 4 system
instead of just base 2. This lead me to the concept of using the two new values
(thats I'm calling "wits") to encode structure, where bits already are used for
encoding value/data. I'm calling this new system "bats" as opposed to "bits".

I was thinking about DNA nucleotides and thinking about what value you could
have with more than binary. I'm a systems thinker-type so I'm really inspired by
ecology and learning from nature. DNA is pretty incredible for what it does and
how it works obviously has some great performance characteristics.

A couple of things that I noticed while I was thinking about it:

1. The four nucleotides are actually two pairs of two. This got me thinking
   about it as a combination of bits (0 | 1 ) and "wits" (< | >) .
2. The nano-machinery that operates on DNA is optimized for linear access and
   transformation and uses codes within the sequence for structure like stop
   codes. Despite our speedups in random memory access over time, it is still
   super slow compared to keeping cache misses down and

So I played around with this for a little bit and realized that I could use the
wits as delimiters around variable length bitstrings. Beyond that, I could use
combinations of wits to create more complex structures.

For example, you can make a variable length 2D array of bitstrings (or bigints
if you prefer to think about them as numbers)

```
>0<1<10<11
>100<101<110<111
>1000<1001<1010<1011<1100<1101<1110<1111
>110011<11001100<1111000011001100
```

Here you can see ">" is used as the start of a sequence, and "<" is used to
extend a sequence with another value. This is "cons"ing a list

```
01 00 00 00 | >0
10 00 00 01 | <1
10 00 00 10 | <10
10 00 00 11 | <11

01 00 01 00 | >100
10 00 01 01 | <101
10 00 01 10 | <110
10 00 01 11 | <111

01 00 10 00 | >1000
10 00 10 01 | <1001
10 00 10 10 | <1010
10 00 10 11 | <1011
10 00 11 00 | <1100
10 00 11 01 | <1101
10 00 11 10 | <1110
10 00 11 11 | <1111

01 11 00 11 | >110011
10 00 11 00 | <001100 //lower 6 bits
00 00 00 11 | 11 //upper 2 bits
```

One easy way of thinking about bats is being represented by two bits:

- 00: 0 (zero) bit

- 01: 1 (one) bit

- 10: < (left) wit

- 11: > (right) wit

So where bits can be either 0 or 1, bats can be 0, 1, <, or >.

## Data-oriented Design

I'm trying to think about software in a data-oriented way. Something that we've
been stuck on for a long time has been fixed size data-structures for the
efficiency of code generation. Structures either have to be a known size in
order to be allocated in memory together, or else the data is scattered across
the heap. My thought is that bats would allow for complex, variable length
structures to be packed together by using a combination of bits to encode the
data and wits to encode the structure.

The way that I'd like to use the wits for structure is fairly simple. In some
ways it is comparable to LISP.

## Sequences

The most basic structure is a sequence. The head of a sequence is signified by a
single > wit. After the head should be a sequence if bits representing the first
value of the sequence. Additional values are added to a sequence using a single
< wit.

For example, the string of bats: >10<0<1<11<101 could be thought of as the
sequence of binary encoded values 2, 0, 1, 3, 5

The basic sequence allows for a compact representation of one or more binary
encoded values, but the structure is linear. A new > wit starts a new sequence.
Sequences cannot nest within sequences. However, with just the > and < wits, you
can easily make 2-dimensional data, similar to a csv. For example:

```
>1<0<0<11>10<1101<1>1>0
```

would be like the equivalent csv:

```csv
1,0,0,3

2,13,1

1

0
```

## Blocks

The next wits construct is called a block. Unlike a sequence which is
1-dimensional, a block is able to contain other wits structures inside of it,
including other blocks.

A block-start is signified with a double-right >> wit sequence. A block-start is
immediately followed by a binary value, which is used as a "tag" for use by
applications. A block tag can be either a single binary value, or multipart
using < wits to extend, just like sequence.

After the block tag, a block can contain 0 or more block elements. A block
element can be a sequence, a block, or a link. (I'll explain links next).

A block ends with a block-end, signified with a double-left << wit sequence. A
block-end is immediately followed by a binary value, which is interpreted as an
integer that indicates how many additional blocks to close. A zero means, "close
the nearest open block". A one means, "close the nearest open block, as well as
next nearest open block". And so on.

Examples:

```
>>1>101<1>1<10<<0
```

Is a block with a tag "1", and contents of two sequences: 5,1 and 1,2.

```
>>1<1>>1>101<<1
```

Is a block with a tag "1,3", and contents of another block with tag "1" and
contents "5", and has a single block-end which closes both blocks

## Links

The final main structure is called a link. This is signified by a triple-right
>>>, and is followed by the link value which can be a single or multi-value,
just like sequences and block tags. This creates a pretty

```
>1<10<11 // sequence [1,2,3]

>>1<10<11 // block-start with tag [1,2,3]

>>>1<10<11 // link to [1,2.3]
```

For both block tags and links, the value is a hierarchical reference that is
able to work like a reference to a symbol table or index. Block tags reference
tag definitions, and links point to blocks somewhere else in the structure.

## Encoding

In order to actually make bats useful, it has to be encoded into an actual
binary format.

### Bats-2x4

The most basic way of encoding it, is something I would call "Bats-2x4". This
encoding uses 2 bits per bat, and 4 bats per byte. It is a byte based encoding.

```
//bats

>101

  

//binary

11 01 00 01
```

It also introduces a need for better compression in the case of large binary
blobs. It should be possible to encode blobs as raw binary without having to
double the space it takes by encoding it in bats. In order to do it, we can use
a new wits operator, "<>".

The <> operator is the "length specified blob" operator. It takes an integer
value "n" and is followed by n bytes that are binary encoded blobs.

Example:

```
>01110<>[16 bytes]

[11 00 01 01] [01 00 10 11][*][*][*][*][*][*][*][*][*][*][*][*][*][*][*][*]
```

### Bats-2:6

I've also been working on an encoding that is slightly more complex, but I think
would also be more efficient. I'm calling it "Bats-2:6" encoding. Instead of
representing each byte as 4 bats, the bats-2:6 encoding tries to optimize for
the most common patterns and getting more bats per byte on average.

The name for the encoding is based on the primary division, the upper two bits
for a wits code, and the remaining 6 bits as bits. This works similarly to
LEB128 as a way to have variable length integers, but instead of just using a
single high bit, this encoding uses the top 2.

In this example, the * signifies a bit that will be used as a binary value

```
00 ****** = ****** (no wits)

01 ****** = <******

10 ****** = >******

11 ****** = uses extended set of codes to interpret the remaining 6 bits

  

extended codes:

00 **** = >>****

01 **** = >>>****

100 *** = >***<>

101 *** = <***<>

110 *** = <<***<>

111000 = <>

111001 = >>< (> + >< an empty head)

111010 = <>< (< + >< an empty cons)

111011 = >>>< (>> + >< a tagless block)
```

The remaining 4 codes are reserved

---

I was thinking about DNA nucleotides and thinking about what value you could
have with more than binary. I'm a systems thinker-type so I'm really inspired by
ecology and learning from nature. DNA is pretty incredible for what it does and
how it works obviously has some great performance characteristics.

A couple of things that I noticed while I was thinking about it:

1. The four nucleotides are actually two pairs of two. This got me thinking
   about it as a combination of bits (0 | 1 ) and "wits" (< | >) .
2. The nano-machinery that operates on DNA is optimized for linear access and
   transformation and uses codes within the sequence for structure like stop
   codes. Despite our speedups in random memory access over time, it is still
   super slow compared to keeping cache misses down and

So I played around with this for a little bit and realized that I could use the
wits as delimiters around variable length bitstrings. Beyond that, I could use
combinations of wits to create more complex structures.

For example, you can make a variable length 2D array of bitstrings (or bigints
if you prefer to think about them as numbers)

```
>0<1<10<11
>100<101<110<111
>1000<1001<1010<1011<1100<1101<1110<1111
>110011<11001100<1111000011001100
```

Here you can see ">" is used as the start of a sequence, and "<" is used to
extend a sequence with another value. This is "cons"ing a list. These sequences
are linear and cannot nest - a sequence ends when another one begins. This
allows for a very compact format for the lowest levels of structure.

```
01 00 00 00 | >0
10 00 00 01 | <1
10 00 00 10 | <10
10 00 00 11 | <11

01 00 01 00 | >100
10 00 01 01 | <101
10 00 01 10 | <110
10 00 01 11 | <111

01 00 10 00 | >1000
10 00 10 01 | <1001
10 00 10 10 | <1010
10 00 10 11 | <1011
10 00 11 00 | <1100
10 00 11 01 | <1101
10 00 11 10 | <1110
10 00 11 11 | <1111

01 11 00 11 | >110011
10 00 11 00 | <001100 //lower 6 bits
00 00 00 11 | 11 //upper 2 bits
```
