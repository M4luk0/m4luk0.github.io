---
title: Command Injection
published: true
---

Hello everyone, today I start a series of blogs where I will explain web vulnerabilities, how to exploit them, fix them and I will leave references where to get more information and practice.

## [](#header-2)Command Injection

Command injection is a vulnerability that allows the attacker to execute OS commands on the web, usually allowing him to control the entire system and to escalate to get into the infrastructure itself through pivoting and other post-exploitation techniques that we will not discuss in this blog.

It takes advantage of the concatenation of commands, depending on the operating system will be one or the other, here is a table:

Unix and Windows:

* &
* &&
* |
* ||

| Unix and Windows | Unix Only             |
|:-----------------|:----------------------|
| &                | ;                     |
| &&               | Newline (0x0a or \n)  |
| |                | ` injected command `  |
| ||               | $( injected command ) |
