# Setup your Ubuntu 20.04 LTS server

## Update and upgrade all services

```{sh}
sudo apt update && sudo apt upgrade -y
```

## Add user and make user admin

```{sh}
sudo adduser <USERNAME>

sudo usermod -a -G sudo <USERNAME>
```

## Disable remote root login

```{sh}
sudo nano /etc/ssh/sshd_config
```

Change this line:

```{txt}
#PermitRootLogin yes
```

To this:

```{txt}
PermitRootLogin no
```

restart sshd:

```{sh}
sudo systemctl restart sshd
```

## Secure Shared Memory

One of the first things you should do is secure the shared memory used on the system. If you're unaware, shared memory can be used in an attack against a running service. Because of this, secure that portion of system memory. You can do this by modifying the /etc/fstab file. Here's how:

Open the file for editing by issuing the command:

```{sh}
sudo nano /etc/fstab
```

Add the following line to the bottom of that file:

```{txt}
tmpfs /run/shm tmpfs defaults,noexec,nosuid 0 0
```

Save and close the file.

> Needs reboot for changes to take effect.

## Install firewall and enable firewall

Firewall let's you explicitly control your ports exposed to the world from your server
It's is always better to not allow remote access to your server through a port if it is not necessary.

```{sh}
sudo apt install ufw

sudo systemctl enable ufw
```

As we will not be using IPV6, we are switching off the IPV6 in the firewall

```{sh}
sudo nano /etc/default/ufw
```

Find IPV6= and set it to no

```{txt}
IPV6=no
```

First things first, setup default policies:

```{sh}
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Allow ssh connection

```{sh}
sudo ufw allow ssh
```

to allow any port:

```{sh}
sudo ufw allow <PORT>/<TCP|UDP|etc. This is optional>
```

Finally let's enable ufw

```{sh}
sudo ufw enable
```

To find out more visit [this page](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-18-04).

## Limit open ports

Most installations of Ubuntu usually have no network services that are listening after the initial install (some hosts may vary). After the server is started, the root user or administrator can define specific services and/or ports to open beyond the defaults.

Testing for open ports can be accomplished using the command netstat -tulpn:

install net-tools:

```{sh}
sudo apt install net-tools
```

test for open ports:

```{sh}
sudo netstat -tulpn
```

## Enable canonical livepatch

install canonical livepatch

```{sh}
sudo snap install canonical-livepatch
```

Go to [canonical livepatch](https://ubuntu.com/livepatch). signup if you are not a cononical or ubuntu user. click on get livepatch and signin.
You will be provided with your livepatch token. Now execute this to enable livepatch

```{sh}
sudo canonical-livepatch enable <YOUR_TOKEN_HERE>
```

## Configure AppArmor for security

Check for anything in complain mode

```{sh}
sudo apparmor_status
```

if nothing is in complain mode you can skip the next part

```{sh}
sudo apt install apparmor-utils
sudo aa-enforce <COMPLAIN_MODE_PATH_OF_SERVICE>
```

## Setup 2FA (Two-Factor Authentication)

For an additional layer of protection, you can also setup Two-Factor Authentication in Ubuntu

> ### Warning:
>
> Be very careful with this setup as you can lock yourself out of the server if set incorrectly.

Follow this [guide to enable 2 factor authentication](https://ubuntu.com/tutorials/configure-ssh-2fa)

## Turn off IPV6

```{sh}
sudo nano /etc/sysctl.conf
```

Add the following lines to the file

```{txt}
net.ipv6.conf.all.disable_ipv6=1
net.ipv6.conf.default.disable_ipv6=1
```

also do edit the following file:

```{sh}
sudo nano /etc/default/grub
```

change/add:

FROM:

```{txt}
GRUB_CMDLINE_LINUX_DEFAULT=""
GRUB_CMDLINE_LINUX=""
```

TO:

```{txt}
GRUB_CMDLINE_LINUX_DEFAULT="ipv6.disable=1"
GRUB_CMDLINE_LINUX="ipv6.disable=1"
```

update your grub boot loader

```{sh}
sudo update-grub
```

## Remove non-secure tools like telnet, ftp, Rsh, etc

```{sh}
sudo apt --purge remove xinetd nis yp-tools tftpd atftpd tftpd-hpa telnetd rsh-server rsh-redone-server
```

TODO: Setup private public key based ssh for all users.

TODO: Disable all password based logins except for any required user.

TODO: Setup strong password policy, password expiry, dictionary for old passwords, etc for password users.

## MySQL setup

### Install MySQL server

```{sh}
sudo apt install mysql-server
```

### Change MySQL Data Directory

Stop MySQL using the following command:

```{sh}
sudo systemctl stop mysql
```

Copy the existing data directory (default located in /var/lib/mysql) using the following command:

```{sh}
sudo cp -R -p /var/lib/mysql /newpath
```

edit the MySQL configuration file with the following command:

```{sh}
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf # or perhaps /etc/mysql/my.cnf
```

Look for the entry for datadir, and change the path (which should be /var/lib/mysql) to the new data directory.

Enter the command:

```{sh}
sudo nano /etc/apparmor.d/usr.sbin.mysqld
```

Look for lines beginning with /var/lib/mysql. Change /var/lib/mysql in the lines with the new path.

Save and close the file.

Restart the AppArmor profiles with the command:

```{sh}
sudo /etc/init.d/apparmor reload
```

Restart MySQL with the command:

```{sh}
sudo systemctl restart mysql
```

Now login to MySQL and you can access the same databases you had before.
