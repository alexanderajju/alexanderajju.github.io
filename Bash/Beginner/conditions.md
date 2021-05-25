---
title: Conditions
layout: page
publish: true
description: Bash Beginner
permalink: bash/conditions
---

Synatx

```bash
#! /bin/bash

if [ conditions ]
then
    statement
fi


```

# Operators

String Comparisons:

| Operator | Usage                                          |
| -------- | ---------------------------------------------- |
| =        | compare if two strings are equal               |
| !=       | compare if two strings are not equal           |
| -n       | evaluate if string length is greater than zero |
| -z       | evaluate if string length is equal to zero     |

Number Comparisons:

| Operator | Usage                                                      |
| -------- | ---------------------------------------------------------- |
| -eq      | compare if two numbers are equal                           |
| -ge      | compare if one number is greater than or equal to a number |
| -le      | compare if one number is less than or equal to a number    |
| -ne      | compare if two numbers are not equal                       |
| -gt      | compare if one number is greater than another number       |
| -lt      | compare if one number is less than another number          |

- Instead of operators , symbols can be also used in inside double parentheses for integers and double square brackets for strings.

```bash

(( 10 > 5 ))

```

Example

```bash

#! /bin/bash

VAR=10

if (( $VAR >= 10 ))
then
  echo "condition satisfied"
fi
```

# if-else

Synatx

```bash
#! /bin/bash

if [ conditions ]
then
    statement
elif [ conditions ]
then
    statement
else
    statement
fi


```

example

```bash

#! /bin/bash

read -p "Enter a number: " VAR

if [ $VAR == 10 ]
then
  echo "condition satisfied"
elif [ $VAR == 11 ]
then
  echo "condition satisfied"
else
  echo "condition not satisfied"

fi


# Output
Enter a number: 10
condition satisfied

Enter a number: 11
condition satisfied

Enter a number: 12
condition  not satisfied

```

[Back](/bash_beginner)
