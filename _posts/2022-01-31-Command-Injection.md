---
title: Command Injection
published: true
---

Hello everyone, today I start a series of blogs where I will explain web vulnerabilities, how to exploit them, fix them and I will leave references where to get more information and practice.

## [](#header-2)What is

Command injection is a vulnerability that allows the attacker to execute OS commands on the web, usually allowing him to control the entire system and to escalate to get into the infrastructure itself through pivoting and other post-exploitation techniques that we will not discuss in this blog.

It takes advantage of the concatenation of commands, depending on the operating system will be one or the other, here is a table:

| Unix and Windows | Unix Only              |
|:-----------------|:-----------------------|
| &                | ;                      |
| &&               | Newline (0x0a or \n)   |
| \|               | \` injected command \` |
| \|\|             | $( injected command )  |

## [](#header-2)PoC

We will be using [DVWA](https://github.com/digininja/DVWA) for this PoC.

DVWA is a vulnerable application for testing to exploit different web vulnerabilities, we will be using it in this blog series.

First, go to "DVWA Security" and set the difficulty to low.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Command_Injection/1.png)

After that, go to command injection and we can start.

It seems that the app does a ping -c 4 to the IP that we put, let's concatenate the whoami command.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Command_Injection/2.png)

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Command_Injection/3.png)

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Command_Injection/4.png)

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Command_Injection/5.png)

We can see that it is vulnerable, from this point we can do reverse shells, etc but we are not going to do them now, in this case, it has shown us the output of the command but let's imagine that it does not, how to detect the vulnerability? by time, we can do a `ping -c 10 127.0.0.1` and it should take 10 seconds to load the web, if so, it is vulnerable, and to see the output we can redirect the command to a file in the following way:

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Command_Injection/6.png)

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Command_Injection/7.png)

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Command_Injection/8.png)



## [](#header-2)How to fix it



## [](#header-2)References



## [](#header-2)Practice

