---
title: Bash Input and variables
layout: pages
publish: true
description: Bash Beginner
permalink: bash_input
---

# Variables

There are no data types. A variable in bash can contain a number, a character, a string of characters.

```bash
#! /bin/bash

STR="Hello World!"
echo $STR

# Output
Hello World!

```

# Input

Input to bash script can be taken using `echo` and `read` command

- echo

```bash
#! /bin/bash

echo -e "Enter your name \c"
read name
echo "$name"

# Output

```

- read

```bash
#! /bin/bash

read -p "Enter your name \c" name
# -p stands for prompt
echo "$name"

# Output

```

-p stands for prompt

```bash
#! /bin/bash

read -sp "Enter your password \c" [assword]
# -s stands for silent
echo "$name"

# Output

```

-s flag stands for silent, which can be used for inputing sensitive inputs like passwords

-a stands for assing vales to array

![Input](/Bash/Beginner/input.gif)

[Back](/bash_beginner)
