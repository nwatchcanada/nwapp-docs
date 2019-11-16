The goal of this article is to provide step-by-step instructions on how to setup [DigitalOcean](https://m.do.co/c/2937038efac6) droplet so we then build our our **NWApp** platform powered by ``CentOS 7``. This is the first tutorial in a series of tutorials.

# Prerequisites

* Create a droplet to your configuration.

* The ``#`` character in ``# yum install -y epel-release`` means that we want to run this script as a ``root`` user account.

* The ``$`` character in ``$ sudo reboot`` means that we want to run this command as a privileged administrator user account.

* From your local developer machine, connect to the newly created server with this command: ```$ ssh -l root nwapp.ca```

# Pre-installation

(1) We want to be using the latest libraries. Read more about [Extra Packages for Enterprise Linux](https://www.tecmint.com/how-to-enable-epel-repository-for-rhel-centos-6-5/).

```bash
# yum install -y epel-release;
```

(2) Update the library.

```bash
# sudo yum -y update;
```

(3) As ``root`` user, copy and paste the following code to install all the libraries we will be using in our project. Other libraries will have individual instructions on how to setup. We are doing this to help speedup the time of installation.

```bash
sudo yum -y install yum-utils;
sudo yum -y groupinstall development;
sudo yum -y install firewalld;
sudo yum -y install ntp;
sudo yum -y install fail2ban;
sudo yum -y install nginx;
sudo yum -y install python36;
sudo yum -y install python36-devel;
sudo yum -y install python-pip;
sudo rpm -Uvh https://download.postgresql.org/pub/repos/yum/10/redhat/rhel-7-x86_64/pgdg-centos10-10-2.noarch.rpm
sudo yum -y install postgresql10-server postgresql10
sudo yum -y install postgis2_10;
sudo yum -y install postgresql10-libs postgresql10-devel postgresql10-plpython
sudo yum -y install redis;
sudo yum -y install yum-cron
sudo yum -y install GraphicsMagick-c++-devel;
sudo yum -y install boost-devel;
```

(3) Update your ``root` password of your droplet. Be sure to set a secure password with the following requirements: (a) 8 characters (b) 1 uppercase character (c) one special character (d) no common words. Finally be sure to not forget this password!

```bash
# passwd root
```

# Secure Root Account and Setup Administrative Account

(1) This section starts off assuming you have completed the above section.

(2) Create our technical operations account which we will use. Afterwords assign a password to that account.

```bash
# adduser techops
# passwd techops
```

(3) Grant administrative privileges to the user.

```bash
# gpasswd -a techops wheel
```

(4) (Optional) **On your local developers machine**, generate the ``ssh`` key pair values. When prompted about passphase, skip it. After generating the key, print the public keys to the console.

```bash
local$ ssh-keygen
local$ cat ~/.ssh/id_rsa.pub
```

(5) Now on your server, copy and paste your ssh key pair into the new user.

```bash
# su - techops
$ mkdir .ssh
$ chmod 700 .ssh
$ vi .ssh/authorized_keys
```

(6) Restrict the permissions of the *authorized_keys* file with this command. Do not skip this command as you will be unable to ``ssh`` into this server without setting the permissions here:

```bash
$ chmod 600 .ssh/authorized_keys
```

(7) On your local developers machine, attempt to log into the server with the ``techops`` user account to confirm it is working. If you cannot log in then please review steps 1 to 7 or search online for answers. Here is an example:

```bash
local$ ssh -l techops 142.93.155.111
```

(8). Exit from the ``techops`` user and disable ``root`` login for the ``sshd`` app.

```bash
$ exit
# vi /etc/ssh/sshd_config
```

(9) Find this line of code and change it to look like this:

```bash
PermitRootLogin no
```

(10) Reload ``sshd`` with our latest change.

```bash
# systemctl reload sshd
```

(11) Now on your local machine, try connecting to the server using the ``root`` account and you'll notice you cannot access it~.

(12) In the future commands, you have to login to the ``techops`` user account and use the ``sudo`` command privilege elevation to use administrative commands.


# Setup Firewall

(1) Then enable the application.

```bash
$ sudo systemctl enable firewalld
$ sudo reboot
```

(2) Log back in from your localmachine.

(3) Start the application.

```bash
$ sudo systemctl start firewalld
```

(4) Confirm it's working.

```bash
$ sudo firewall-cmd --state
```

(5) Add rules so ``nginx`` will be able to accessible by the the internet.

```
$ sudo firewall-cmd --permanent --add-service=http
$ sudo firewall-cmd --permanent --add-service=https
$ sudo firewall-cmd --reload
$ sudo firewall-cmd --runtime-to-permanent
```

(6) Would you like to know more? [Learn more](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-using-firewalld-on-centos-7). It is encouraged to read this article to understand the next few steps made.


### Adjust Timezone

(1) List what available timezones there are.

```bash
$ sudo timedatectl list-timezones
```

(2) Set ``America/Toronto`` timezone for our OS build.

```bash
$ sudo timedatectl set-timezone America/Toronto
```

(3) Make the timezone change permanent.

```bash
$ sudo timedatectl
```

(4) Start ``ntp`` app, and enable it to start at boot-time.

```bash
$ sudo systemctl start ntpd
$ sudo systemctl enable ntpd
```

### Auto-Update Cron

* https://www.techrepublic.com/article/how-to-enable-automatic-security-updates-on-centos-7-with-yum-cron/

(1) Enable and start our cron.

```bash
$ sudo systemctl start yum-cron;
$ ​sudo systemctl enable yum-cron;
```

(2) Confirm that our installation was a successs.

```bash
$ systemctl status  yum-cron.service
```

(3) Look at our configuration:

```bash
$ sudo vi /etc/yum/yum-cron.conf
```

(4) Next, locate the line:

```
apply_updates = no
```

(5) And change to:

```
apply_updates = yes
```

(6) Restart the service.

```bash
$ sudo systemctl restart yum-cron
```

(7) Confirm that our ``cron`` service is running.

```bash
$ systemctl status  yum-cron.service
```

# Setup Fail2ban

(1) Enable the service to start at boot-time.

```bash
$ sudo systemctl enable fail2ban
```

(2) Setup our configuration. Start by opening up our configuration.

```bash
$ sudo vi /etc/fail2ban/jail.local
```

(3) Copy and paste.

```
[DEFAULT]
## Ban hosts for one hour:
#bantime = 7200
# Ban host for one day:
bantime = 172800
ignoreip = 127.0.0.1/8

# Override /etc/fail2ban/jail.d/00-firewalld.conf:
banaction = firewallcmd-ipset

[sshd]
enabled = true
```

(4) Start the service ``fail2ban`` app.

```bash
$ sudo systemctl start fail2ban
```

(5) If you get any problems, please run the following and google a solution. Special thanks to [this article](https://askubuntu.com/questions/1099306/job-for-fail2ban-service-failed-because-the-control-process-exited-with-error).

```bash
sudo fail2ban-client -vvv -x start
```

(6) Check to confirm that the ``fail2ban`` app has been successfully installed and is currently running in our build.

```bash
$ sudo fail2ban-client status
```

(7) Would you like to know more? [Read more here](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-centos-7).


#Setup Nginx

(1) Start ``nginx``.

```bash
$ sudo systemctl start nginx
```

(2) Enable ``nginx`` to startup on boot-time. Use the following command to do so:

```bash
$ sudo systemctl enable nginx
```

(3) Confirm our server works. Open in a web browser:

```bash
http://nwapp.ws
```

(4) (Optional) Here are the ``nginx`` important files and directories to take note of:

  * The default server root directory (top level directory containing configuration files): **/etc/nginx**
  * The main Nginx configuration file: **/etc/nginx/nginx.conf**
  * Server block (virtual hosts) configurations can be added in: **/etc/nginx/conf.d**
  * The default server document root directory (contains web files): **/usr/share/nginx/html**
  * The default log for errors: **/var/log/nginx/error.log**
  * The default log for access: **/var/log/nginx/access.log**


# Setup Python
1. Install our Python dependencies.

        sudo yum -y install python36;
        sudo yum -y install python36-devel;
        sudo yum -y install python-pip;

2. Confirm you installed the correct library.

        python3.6 -V

3. We will next install pip, which will manage software packages for Python:

        curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"
        sudo python3.6 get-pip.py
        sudo rm get-pip.py
        sudo pip install --upgrade pip
        sudo ln -s /usr/local/bin/pip3.6 /bin/pip3.6

4. Confirm we have the proper version.

        pip -V

5. Install some dependent libraries.

        sudo pip-3.6 install virtualenv
        sudo ln -s /usr/local/bin/virtualenv /bin/virtualenv


# Postgres
(1) Initialize our database.

```bash
$ sudo /usr/pgsql-10/bin/postgresql-10-setup initdb
```

(2) Open the HBA configuration with your favorite text editor. We will use vi:

```bash
$ sudo vi /var/lib/pgsql/10/data/pg_hba.conf
```

(3) Find the lines that looks like this, near the bottom of the file:

```
host    all             all             127.0.0.1/32            ident
host    all             all             ::1/128                 ident
```

(4) Then replace "ident" with "md5", so they look like this:

```
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```

(5) Open the configuration with your favorite text editor. We will use vi:

```
$ sudo vi /var/lib/pgsql/10/data/postgresql.conf
```

(6) Then modify the following to look like this:

```
listen_addresses = '*'
```

(7) Start the service

```
$ sudo systemctl start postgresql-10
```

(8) Make sure it boots all the time.

```
$ sudo systemctl enable postgresql-10
```

(9) Begin using it...

```
$ sudo -i -u postgres
$ psql
```

(10) Would you like to know more about PostGres setup? [Learn more.](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-centos-7)

# Setup Redis
https://www.linode.com/docs/databases/redis/deploy-redis-on-centos-7

```bash
$ sudo systemctl start redis
$ sudo systemctl enable redis
```

You can edit.

```bash
$ sudo vi /etc/redis.conf
```

# NodeJS, NPM, React

1. Install ``Node``.

    ```
    curl -sL https://rpm.nodesource.com/setup_10.x | sudo bash -
    sudo yum -y install nodejs
    ```

2. Check version.

    ```
    node --version
    ```

3. Install ``npm``.

    ```
    sudo npm install -g npm@next
    ```

4. (OPTIONAL) You can disable ``https`` and use unsecure ``http`` by running the following. This step was provided by [this link](https://stackoverflow.com/a/13119867).

    ```
    npm config set registry http://registry.npmjs.org/
    ```

4. Check version.

    ```
    npm --version
    ```

5. Install ``react``.

    ```
    sudo npm install -g create-react-app
    ```

6. Check version.

    ```
    create-react-app --version
    ```

7. Would you like to [learn more](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-a-centos-7-server)?
