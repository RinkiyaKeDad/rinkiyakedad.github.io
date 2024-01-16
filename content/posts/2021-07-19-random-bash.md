---
title: "Random Bash"
read_time: true
date: 2021-07-19T20:12:05+05:30
tags:
  - bash
  - linux
---

Hey everyone, recently I developed quite an interest in Bash so I've been trying to learn it. Unlike other things, this time it wasn't much-structured learning rather just some random tidbits I picked on here and there. In this post I'll share those things so that together we can demystify this fancy language. Also, some things might just be about Linux in general rather than bash specific.

## #!/bin/bash

You might have seen that a lot of bash scripts begin with `#!/bin/bash`. So what exactly is this?

`#!` is called shebang. It gets its name because it is a combination of "sharp" (`#`) and "bang" (`!`). We start each script with a shebang and then the path to the interpreter.  This tells the shell what program to interpret the script with. In this case, we want the script to be run by the `bash` interpreter so we specify the path to it.

## Something Interesting About `ls -l`

When you do `ls -l` you see something like this 

```
-rw-r--r-- 1 arsh staff 19 Jul 21 11:57 file.sh
```

Let us break the first "word" and see what it means. Even before we do that here's something you should know to understand what comes next,

```
r -> read
w -> write
x -> execute
```

You'll notice that removing the first dash there are three sets each with three characters. Each of the sets in order of left to the right refers to permissions granted to the owner of the file, to the group of the file, and permissions available to everyone else on the system of the file respectively.

The permissions are read as `rwx` and if either one of them is a `-` then that means that particular permission is not available to that user. For example here, it would mean that the owner of the file has read and write permissions (`rw-`) where as the group of the file and everyone else has only read permissions (`r--`). 

## Changing permissions which `chmod`
If you want to change the permissions of a file (which we saw using `ls -l`) you use `chmod`. If you want the file to be executable by everyone but give write permissions to only the owner you can run, `chmod 755 <filelocation>`. But where does this `755` come from?

To understand that first take a note of these values

```
r (read) -> 4
w (write) -> 2
x (execute) -> 1
```

Now let's try to see a pattern :)

```
7 -> 4 + 2 + 1
5 -> 4 + 0 + 1
5 -> 4 + 0 + 1
```

Starting to make sense? The first row corresponds to the permissions the owner of the file would have, the second corresponds to the ones the group will have, and the third, corresponds to the permissions everyone else on the filesystem will have. You can create similar combinations as per your need to modify the file permissions. 

## Some Basics About Variables
Variables in bash can be directly assigned values without specifying any type like int, string, etc. 

```
VAR_NAME="value"
VAR_NAME=5
```

To reference a variable we use a dollar sign along with its name, for example, `$VAR_NAME`. 

Single quotes prevent the expansion of variables whereas double don't. For example,

```
echo '$VAR_NAME is a variable' -> $VAR_NAME is a variable
echo "$VAR_NAME is a variable" -> value is a variable (if VAR_NAME was set to "value")
```

We can also place the variable name in `{}` and precede the opening brace with `$`. This is useful in cases like,
```
WORD="talk"
echo "${WORD}ing"
```
Output -> "talking"

Also, note that variable names are in upper case by **convention** but this isn't mandatory by bash.

## Something About Commands
If you're not sure about some command, let's say `head` you can run `type -a head` and you would see something like 
```
head is /usr/bin/head
```
On the other hand for some commands like `echo` you would see something like,
```
echo is a shell builtin
echo is /bin/echo
```
If a command is a shell builtin you can use `help <command>` to get information about it. `help` is a bash command which uses internal bash structures to store and retrieve information about bash commands. 

If something isn't a shell builtin, you can use the `man` command to get information about it. Quoting directly from [here](https://unix.stackexchange.com/a/86584),

> "Help is a built-in "usage" of the command, and not all commands implement it, or at least not the same way, however, man is a command by itself which is a pager program that reads manual."

Every time we type something in the command line, bash first tries to find a function with that name to execute, if not found then it looks for that command in its list of built-in commands, if it's not found even there then bash searches through the list of dirs defined in the `$PATH` variable and executes the first match it finds. If still no match then it says command not found. 

You can see the list of dirs it will search through by running `echo $PATH`. You can also edit this `$PATH` to add more dirs where it can look for commands. It goes through the colon-separated dirs defined in the `$PATH` in order and as soon as it finds a match it stops there. If you want to see what gets executed when you run a command, you can run `which <command>`
```
which depstat                                                      
/Users/arsh/go/bin/depstat
```
This means instead of running [depstat](https://github.com/kubernetes-sigs/depstat) I could also have run `/Users/arsh/go/bin/depstat`. If you do `which -a <command>` then it will show you all instances it found of that command.

Another thing about commands is that you can combine their shorthands for flags. For example, `id -n -u` can also be executed as `id -nu` or `id -un`. 

One last thing is that you can store the output of commands in a variable using the following syntax,
```
USER_NAME=$(id -un)
```
Also, note that if the exit status of a command is 0 that means it executed successfully without any errors. Non-zero exit status always means something went wrong.

Well this was it for this article and I hope you learned some interesting linux-y things :)