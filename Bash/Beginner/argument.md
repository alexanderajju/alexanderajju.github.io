---
title: Argument passing
layout: page
publish: true
description: Bash Beginner
permalink: bash_arguments
---

To pass any number of arguments to the bash function simply put them right after the function's name, separated by a space. It is a good practice to double-quote the arguments to avoid the misparsing of an argument with spaces in it. The passed parameters are `$1` , `$2` , `$3`

# First method

```bash

#! /bin/bash

echo $1 $2 $3 'echo $1 $2 $3'


```

```bash

┌──(aju㉿ParrotOs)-[~/Aju/bash__scripiting]
└─$ ./arguments.sh 1 2 3

# Output

1 2 3 echo $1 $2 $3

```

# Second Method

- Using array

```bash

#! /bin/bash

args=("$@")

echo "${args[@]}


```

```bash

┌──(aju㉿ParrotOs)-[~/Aju/bash__scripiting]
└─$ ./arguments.sh 1 2 3

# Output

1 2 3

```

- total number of arguments passed by the user can be displayed by passing `$@` in `echo`.

# Third Method

- Using of flags. This method can be implemented using `case`, `getopts` loops.

![Arguments](/Bash/Beginner/arguments.gif)

[Back](/bash_beginner)
