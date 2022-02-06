---
title: File Inclusion
published: true
---

## [](#header-2)What is

File Inclusion is a vulnerability with two variants, local and remote, the local one allows the attacker to read files from the system and the remote one allows the attacker to read files from the internet, we will see the usefulness of the latter later.

## [](#header-2)Local File Inclusion PoC

We will be using [DVWA](https://github.com/digininja/DVWA) for this PoC.

DVWA is a vulnerable application for testing to exploit different web vulnerabilities, we will be using it in this blog series.

First, go to "DVWA Security" and set the difficulty to low.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Command_Injection/1.png)

After that, go to file inclusion and we can start.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/File_Inclusion/1.png)

We see that there are 3 files, let's click on one and look at the url.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/File_Inclusion/2.png)

Now, let's try to read /etc/passwd like this.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/File_Inclusion/3.png)

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/File_Inclusion/4.png)

Why is it read? easy, what we are doing is going back directories with ../ and then we only have to go to the place where the file is, in this case /etc/passwd; you have to put many ../ because even if it reaches the root directory it will not fail or go further back since it is the furthest back directory; so, the more ../ the better.

This can be protected with filters and so on but there are also bypasses of these filters although I will not explain them here, I will leave in the References section websites where you can learn these bypasses.

## [](#header-2)Remote File Inclusion PoC

We do the same as in the previous PoC, we go to File Inclusion and select a file, but this time, instead of a local file, we will read a remote file and get access to the machine by doing a reverse shell.

You may ask "How by reading a file can we have a shell?!" the web runs php, so if we read a php reverse shell it will run, let's do it.

We will be using the [Pentest monkey php reverse shell](https://github.com/pentestmonkey/php-reverse-shell).

We have to change the reverse shell parameters of the ip and port to set our own.

Once the reverse is ready, we must make a server in the directory where we have the reverse shell with

```shell
python3 -m http.server
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/File_Inclusion/5.png)

Now we are listening on the port indicated in the reverse.

```shell
nc -lvp PORT
```
![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/File_Inclusion/8.png)

Finally, we go to the web, put our web with the file in the url and execute.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/File_Inclusion/6.png)

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/File_Inclusion/7.png)

And now that you have read our reverse and gained access to the system, the remote mode also has filters and bypasses but we will not deal with them here.

## [](#header-2)How to fix it

We can create a whitelist with allowed values.

After receiving the file to be read, let the application make sure that the file is from the directory where the readable files are located.

## [](#header-2)References

https://portswigger.net/web-security/file-path-traversal

https://book.hacktricks.xyz/pentesting-web/file-inclusion

https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion

## [](#header-2)Practice

https://portswigger.net/web-security/file-path-traversal
