---
title: Brainpan Tryhackme Writeup
published: true
---

Today I bring you the writeup of [Brainpan](https://tryhackme.com/room/brainpan), a tryhackme machine focused on Buffer Overflow, here we go!

First we start with a port scan to see which ports are open.

```shell
nmap -sC -sV -p- --min-rate 5000 IP
```

With -p- we indicate that we want it to look at all ports,  -sC we indicate that we want to run the default nmap scripts, -sV to get the service versions and the --min-rate 5000 to do it very fast.

FOTO

It has two open ports, one web and something we don't know what it is, let's connect via netcat to that port.

```shell
nc IP 9999
```

FOTO

It looks like the program with which we have to do the BoF; now let's go to the web.

FOTO

There doesn't seem to be much, let's do some directory fuzzing to see if there is anything we don't see.

```shell
ffuf -w WORDLIST:FUZZ -u http://IP/FUZZ -t THREADS
```

Where it says WORDLIST put the path of the wordlist, in IP put the IP of the target and in THREADS the threads that we want to put, the more threads, the faster.

We get a /bin, let's see what it is.

FOTO

Let's check which architecture it is to open it with x32 or x64

FOTO 6

Ok, looks like the 9999 port binary to exploit it locally before going to the app; let's download it and exploit it on windows with [x64dbg](https://github.com/therealdreg/x64dbg-exploiting/releases/download/1.1/x64dbg-exploitingv1.1.zip), by the way, I have several [demos](https://github.com/therealdreg/x64dbg-exploiting) made in the official repo.

To install the x64dbg with plugins and so on we have to follow the [instructions](https://github.com/therealdreg/x64dbg-exploiting) of the official repo.

FOTO 7-11

Make sure to create the logs folder inside C:

FOTO 5

To configure the logs folder we have to put the following:

```shell
mona.mona("config -set workingfolder c:\\logs\\%p")
```

FOTO

If the previous commands were successful, it should look like this

FOTO

Now hit File->Open and select the vulnerable program

FOTO

FOTO

Click two times on the Run icon to open the program

FOTO

FOTO

Perfect, let's start with the attack, first we must find out how many characters we have to put to overflow the stack; to do this, without leaving the log we put the following command:

```shell
mona.mona("pattern_create 1000")
```

FOTO

Copy the output of the command and paste it into a .txt file

FOTO

Now let's start the script to do our local testing:

```python

```
