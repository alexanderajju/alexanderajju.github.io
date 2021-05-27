---
title: File Operations
layout: page
publish: true
description: Bash Beginner
permalink: bash/file_operations
---

In this tutorial we will learn about file operators in Shell Programming by using the [if statement](/bash/conditions) to perform tests

| Operator | Note                                         |
| -------- | -------------------------------------------- |
| -e       | To check if the file exists.                 |
| -r       | To check if the file is readable.            |
| -w       | To check if the file is writable.            |
| -x       | To check if the file is executable.          |
| -s       | To check if the file size is greater than 0. |
| -d       | To check if the file is a directory.         |

```bash

#! /bin/bash
 echo -e "enter the name of the file : \c"
 read file_name

if [ -e $file_name ]
 then
   echo "$file_name exists"
  else
    echo "$file_name Not found"
fi


```

![File Operations](/Bash/Beginner/file.gif)

[Back](/bash_beginner)
