---
layout: post
title:  "Linux Alsa API"
author: Willen
categories: [ sound ]
tags: [ ALSA ]
comments: false
rating: false
---

# Linux ALSA API

# ALSA Library API

The ALSA library API is the interface to the ALSA drivers. Developers need to use the functions in this API to achieve native ALSA support for their applications. The ALSA lib documentation is a valuable developer reference to the available functions. In many ways it is a tutorial. The latest on-line documentation is generated from the alsa-lib GIT sources.

- [ALSA library API reference](http://www.alsa-project.org/alsa-doc/alsa-lib/)

The currently designed interfaces are listed below:

1. Information Interface (/proc/asound)
2. Control Interface (/dev/snd/controlCX)
3. Mixer Interface (/dev/snd/mixerCXDX)
4. PCM Interface (/dev/snd/pcmCXDX)
5. Raw MIDI Interface (/dev/snd/midiCXDX)
6. Sequencer Interface (/dev/snd/seq)
7. Timer Interface (/dev/snd/timer)

You can also generate it yourself. Install the [doxygen tool](http://www.doxygen.org/) and type 'make doc' in the alsa-lib directory.

There is a stripped version of ALSA-library for small systems like embedded devices, [SALSA-Library](https://www.alsa-project.org/wiki/SALSA-Library). It's designed to be source-level API compatible with the normal ALSA library, but has no binary compatibility.

- [SALSA-Library](https://www.alsa-project.org/wiki/SALSA-Library)

Some aspects of the library operation are affected by [environment variables](https://www.alsa-project.org/wiki/LibEnvVars)



Configuration files use a simple format allowing modern data description like nesting and array assignments.

# Whitespace 空格

Whitespace is the collective name given to spaces (blanks), horizontal and vertical tabs, newline characters, and comments. Whitespace can indicate where configuration tokens start and end, but beyond this function, any surplus whitespace is discarded. For example, the two sequences

a 1 b 2

and

a 1 

   b 2

are lexically equivalent and parse identically to give the four tokens:

a

1

b

2

The ASCII characters representing whitespace can occur within literal strings, in which case they are protected from the normal parsing process (they remain as part of the string). For example:

name "John Smith"

parses to two tokens, including the single literal-string token "John Smith".

# Line continuation with

A special case occurs if a newline character in a string is preceded by a backslash (). The backslash and the new line are both discarded, allowing two physical lines of text to be treated as one unit.

"John \

Smith"

is parsed as "John Smith".

# Comments

A single-line comment begins with the character #. The comment can start at any position, and extends to the end of the line.

a 1  # this is a comment

# Including configuration files

To include another configuration file, write the file name in angle brackets. The prefix `confdir:` will reference the global configuration directory.

</etc/alsa1.conf>

<confdir:pcm/surround.conf>

# Punctuators 标点器

The configuration punctuators (also known as separators) are:

{} [] , ; = . ' " new-line form-feed carriage-return whitespace

## Braces 大括号

Opening and closing braces { } indicate the start and end of a compound statement:

a {

  b 1

}

## Brackets 括号

Opening and closing brackets indicate a single array definition. The identifiers are automatically generated starting with zero.

a [

  "first"

  "second"

]

The above code is equal to

a.0 "first"

a.1 "second"

## Comma and semicolon 逗号和分号

The comma (,) or semicolon (;) can separate value assignments. It is not strictly required to use these separators because whitespace suffices to separate tokens.

a 1;

b 1,

## Equal sign

The equal sign (=) can separate variable declarations from initialization lists:

a=1

b=2

Using equal signs is not required because whitespace suffices to separate tokens.

# Assignments

The configuration file defines id (key) and value pairs. The id (key) can be composed from ASCII digits, characters from a to z and A to Z, and the underscore (_). The value can be either a string, an integer, a real number, or a compound statement.

## Single assignments

a 1 # is equal to

a=1 # is equal to

a=1;    # is equal to

a 1,

## Compound assignments (definitions using braces)

a {

  b = 1

}

a={

  b 1,

}

# Compound assignments (one key definitions)

a.b 1

a.b=1

## Array assignments (definitions using brackets)

a [

  "first"

  "second"

]

## Array assignments (one key definitions)

a.0 "first"

a.1 "second"

# Operation modes for parsing nodes 解析节点的操作模式

By default, the node operation mode is 'merge+create', i.e., if a configuration node is not present a new one is created, otherwise the latest assignment is merged (if possible - type checking). The 'merge+create' operation mode is specified with the prefix character plus (+).

The operation mode 'merge' merges the node with the old one (which must exist). Type checking is done, so strings cannot be assigned to integers and so on. This mode is specified with the prefix character minus (-).

The operation mode 'do not override' ignores a new configuration node if a configuration node with the same name exists. This mode is specified with the prefix character question mark (?).

The operation mode 'override' always overrides the old configuration node with new contents. This mode is specified with the prefix character exclamation mark (!).

defaults.pcm.!device 1

# Syntax summary

\# Configuration file syntax

\# Include a new configuration file

<filename>

\# Simple assignment

name [=] value [,|;]

\# Compound assignment (first style)

name [=] {

​        name1 [=] value [,|;]

​        ...

}

\# Compound assignment (second style)

name.name1 [=] value [,|;]

\# Array assignment (first style)

name [

​        value0 [,|;]

​        value1 [,|;]

​        ...

]

\# Array assignment (second style)

name.0 [=] value0 [,|;]

name.1 [=] value1 [,|;]