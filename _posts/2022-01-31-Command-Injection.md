---
title: Command Injection
published: true
---

Hello everyone, today I start a series of blogs where I will explain web vulnerabilities, how to exploit them, fix them and I will leave references where to get more information and practice.

## [](#header-2)What is

Command injection is a vulnerability that allows the attacker to execute OS commands on the web, usually allowing him to control the entire system and to escalate to get into the infrastructure itself through pivoting and other post-exploitation techniques that we will not discuss in this blog.

It takes advantage of the concatenation of commands, depending on the operating system will be one or the other, here is a table:

| Unix and Windows                                    | Unix Only                                           |
|:----------------------------------------------------|:----------------------------------------------------|
| & (Executes all commands regardless                 | ; (Executes all commands regardless                 |
|    of whether they were performed correctly or not) |    of whether they were performed correctly or not) |
| && (Execute the second command if                   |                                                     |
|     the first command was executed correctly.)      | Newline (0x0a or \n)                                |
| \| (Pipe, it takes the output of the first          | \` injected command \`                              |
|     command and passes it to the input of the       |                                                     |
|     second, but if the second command does not      |                                                     |
|     take input it executes it anyway although it    |                                                     |
|     does not display it.)                           |                                                     |
| \|\| (Executes either the first command or the      | $ ( injected command )                              |
|       second, i.e., if the first command is not     |                                                     |
|       executed correctly, it will execute the       |                                                     |
|       second command.)                              |                                                     |

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

Before moving on to the next section I will leave you with a table of useful commands for this vulnerability:

| Purpose of command    | Linux            | Windows       |
|:----------------------|:-----------------|:--------------|
| Name of current user  | whoami           | whoami        |
| Operating system      | uname -a         | ver           |
| Network configuration | ifconfig or ip a | ipconfig /all |
| Network connections   | netstat -an      | netstat -an   |
| Running processes     | ps -ef           | tasklist      |

## [](#header-2)How to fix it

The best way to fix this is for the app not to execute system commands, there are other ways to implement the functionality we want through APIs.

In case we can't do without this function we can:
* Whitelist of allowed values.
* Validate only the values we need, i.e., if it is a ping, only allow numbers.

## [](#header-2)References

https://portswigger.net/web-security/os-command-injection

https://book.hacktricks.xyz/pentesting-web/command-injection

https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection

## [](#header-2)Practice

https://portswigger.net/web-security/os-command-injection

https://www.root-me.org/es/Desafios/Web-Servidor/Command-injection-Filter-bypass
