---
title: How to install BurpSuite in Debian
published: true
---

In this blog, I will show you how to install Burp Suite community on debian 11, let's get started.

## [](#header-2)Prerequisites

In order to install Burp, we need a version of java and Burp itself, let's start with java.

To install the required java version we put

```shell
sudo apt install default-jre
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Burp_install/1.png)

To install [Burp](https://portswigger.net/burp/releases/professional-community-2021-12-1?requestededition=community) go to their website and download the one that suits us with our pc.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Burp_install/2.png)

Now go to where Burp was downloaded and run it.

```shell
bash burpsuite_community_linux_v2021_12_1.sh
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Burp_install/3.png)

In the installation, click "Next" until the download is complete.

Now that we have Burp downloaded, we open it and create a temporary project, click "next" and then "start burp".

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Burp_install/4.png)

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Burp_install/5.png)

To use it we go to proxy and click on "open browser", I recommend using that browser and not foxy proxy as burp works better with its own browser.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Burp_install/6.png)

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/Burp_install/7.png)

And that's it! we can now use burp.
