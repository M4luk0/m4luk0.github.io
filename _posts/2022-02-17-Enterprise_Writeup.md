---
title: Enterprise Tryhackme Writeup
published: true
---

Today I bring you the writeup of [Enterprise](https://tryhackme.com/room/enterprise), a tryhackme machine focused on Active Directory exploitation, here we go!

First we start with a port scan to see which ports are open.

```shell
nmap -p- --min-rate 5000 IP -Pn
```

With -p- we indicate that we want it to look at all ports, -Pn is not to make ping requests since it is a windows and blocks them and the --min-rate 5000 to do it very fast.

FOTO

Now that we know the ports, let's scan them deeply to see versions and so on.

```shell
nmap -sC -sV -p53,80,88,135,139,389,445,464,593,636,3268,3269,3389,5357,5985,7990,9389,47001,49664,49665,49666,49668,49669,49670,49672,49676,49701,49710,49833 --min-rate 5000 IP -Pn
```

With -sC we indicate that we want to run the default nmap scripts and -sV to get the service versions.

FOTO

Now, let's add to /etc/hosts the domain name.

```shell
IP lab.enterprise.thm
```

FOTO 7

After a long time listing the different services of the machine we found this web where we are told that they are passed to [github](https://github.com/Enterprise-THM), let's check this github.

FOTO

FOTO

We see a [person](https://github.com/Nik-enterprise-dev) on that github, let's take a look at theirs.

FOTO

It has a repository, if we look at the [history](https://github.com/Nik-enterprise-dev/mgmtScript.ps1/commit/bc40c9f237bfbe7be7181e82bebe7c0087eb7ed8) of that repo we find a username and password.

FOTO

With the information we have and after listing several services with that name and password, we are going to use the impacket module GetUsersSPNs

This module will try to find Service Principal Names that are associated with normal user account.
#   Since normal account's password tend to be shorter than machine accounts, and knowing that a TGS request
#   will encrypt the ticket with the account the SPN is running under, this could be used for an offline
#   bruteforcing attack of the SPNs account NTLM hash if we can gather valid TGS for those SPNs.

```shell
GetUserSPNs.py lab.enterprise.thm/nik:ToastyBoi!
```

FOTO

We have found a service called bitbucket, we are going to take its NTLM hash and then crack it.

```shell
GetUserSPNs.py lab.enterprise.thm/nik:ToastyBoi! -request
```

FOTO

