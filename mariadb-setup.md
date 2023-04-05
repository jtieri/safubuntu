# MariaDB Setup

We will install mariadb and go through the basic installation process 

Install

```bash
sudo apt install mariadb-server

```

Start

```bash
sudo systemctl start mariadb.service

```

Setup

```bash
sudo mysql_secure_installation
```

Enter root passphrase when prompted
Keep root password default

On Ubuntu, the root account for MariaDB is tied closely to automated system maintenance, so we should not change the configured authentication methods for that account. Doing so would make it possible for a package update to break the database system by removing access to the administrative account. Type N and then press ENTER.

Create priveleged user account

```bash
sudo mariadb
```

Then create a new user with root privileges and password-based access. Be sure to change the username and password to match your preferences:

```bash
GRANT ALL ON *.* TO 'admin'@'localhost' IDENTIFIED BY 'password' WITH GRANT OPTION;
```

Flush the privileges to ensure that they are saved and available in the current session:

```bash
FLUSH PRIVILEGES;
```

Following this, exit the MariaDB shell:

```bash
exit
```

Login to test that everything is working properly

```bash
mysqladmin -u admin -p version
```