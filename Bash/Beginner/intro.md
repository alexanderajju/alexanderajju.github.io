---
title: Introduction
layout: pages
publish: true
description: Bash Beginner
permalink: bash_intro
---


A shell script is a computer program designed to be run by the Unix shell, a command-line interpreter. The various dialects of shell scripts are considered to be scripting languages. Typical operations performed by shell scripts include file manipulation, program execution, and printing text.

- **This Tutorial is mainly focused for users using Linux based Operating system, Mac and Windows 10**

# Let's Begin

- Operating Sytem(mine)- Parrot Security
- Editor(mine) - vim

Any editor can be used for this tutorials its fully upto you guys. [VisalStudio code](https://code.visualstudio.com) is recommended for beiginners. In bash scripting any linux terminal commands can be used

In order to get start we must add following to our file's first line `#!/bin/bash`

Adding `#!/bin/bash` as the first line of our script, tells the OS to invoke the specified shell to execute the commands that follow in the script. `#!` is often referred to as a "hash-bang", "she-bang" or "sha-bang"


```bash
#!/bin/bash

```

# Hello world

Let's begin our first script  by creating a hello world. Move to favorable directory

- Create a file named `hello.sh`

- Add the following in to the file and save the file

```bash
#!/bin/bash

echo "Hello World!"

# echo command is used to display line of text/string that are passed as an argument

```
- To execute the script open the terminal in the directory where `hello.sh` is

- Give executable permission to `hello.sh`

```

chmod +x hello.sh

# chmod is the command used to change the access permissions of file systemS

```
- Then to execute the file do following add a period and forward slash + file name

```bash

./hello.sh

# Output
Hello World!

```

[Back](/bash_beginner)




