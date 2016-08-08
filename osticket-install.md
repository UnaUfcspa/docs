# Instalação do OSTicket 1.9.12


## Setup

Para o funcionamento do OSTicket é necessário ter um servidor *http* e um *sgbd*. Neste caso utilizaremos o **httpd** juntamente com o **MariaDB**.
```sh
[user@localhost ~]$ sudo yum install httpd mariadb-server
[user@localhost ~]$ sudo firewall-cmd --zone=public --add-port=http/tcp
[user@localhost ~]$ sudo firewall-cmd --zone=public --add-port=http/tcp --permanent
[user@localhost ~]$ sudo systemctl enable httpd mariadb-server
[user@localhost ~]$ sudo systemctl restart httpd mariadb-server
```

Após a instalação do **MariaDB** é preciso setar algumas opções de segurança do serviço, para isto basta executar o seguinte:

```sh
[user@localhost ~]$ mysql_secure_installation
/bin/mysql_secure_installation: line 379: find_mysql_client: command no
t found

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
[user@localhost ~]$
```

Verifique se o horário do servidor já está corretamente configurado com
a timezone correta e também se o **ntp** está instalado.

```sh
[user@localhost ~]$ timedatectl
      Local time: Thu 2016-08-04 09:50:48 BRT
  Universal time: Thu 2016-08-04 12:50:48 UTC
        RTC time: Sat 2016-08-06 07:04:41
       Time zone: America/Sao_Paulo (BRT, -0300)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: no
      DST active: no
 Last DST change: DST ended at
                  Sat 2016-02-20 23:59:59 BRST
                  Sat 2016-02-20 23:00:00 BRT
 Next DST change: DST begins (the clock jumps one hour forward) at
                  Sat 2016-10-15 23:59:59 BRT
                  Sun 2016-10-16 01:00:00 BRST
```
A configuração da timezone correta é realizada pelo arquivo
*/etc/localtime*. Realizar um backup do arquivo original não é necessário,
mas é uma boa ideia.

```sh
[user@localhost ~]$ sudo cp /etc/localtime /etc/localtime.backup
[user@localhost ~]$ sudo ln -s /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime 
```
O **ntpd** pode ser instalado diretamente pelo *yum*.

```sh
[user@localhost ~]$ sudo yum install ntp
[user@localhost ~]$ sudo systemctl enable ntpd
[user@localhost ~]$ sudo systemctl restart ntpd
```

Habilitar o repositório de pacotes extras do Centos
```sh
[user@localhost ~]$ sudo yum install epel-release
```

## Instalando o PHP

```sh
[user@localhost ~]$ sudo yum install php php-mysql php-gd php-mbstring php-xml php-imap php-php-gettext php-JsonSchema
```
Configure a timezone correta no arquivo */etc/php.ini*


```apache
[Date]
; Defines the default timezone used by the date functions
; http://php.net/date.timezone
date.timezone = "America/Sao_Paulo"
```

```sh
[user@localhost ~]$ sudo systemctl restart httpd
```

## Configurando o SELinux

Para que tudo funcione corretamente com o SELinux ativo é necessário que
ativemos algumas de suas *flags* de operação relativas ao **httpd**.

```sh
[user@localhost ~]$ sudo setsebool -P httpd_can_sendmail 1
[user@localhost ~]$ sudo setsebool -P httpd_can_network_connect 1
[user@localhost ~]$ sudo setsebool -P httpd_read_user_content 1
[user@localhost ~]$ sudo setsebool -P httpd_unified 1
```

## Download dos arquivos do OSTicket

Instalar o Unzip
```sh
[user@localhost ~]$ sudo yum install unzip
```
Os arquivos devem ser obtidos diretamente no site oficial do OSTicket.

```sh
[user@localhost ~]$ cd /var/www/html 
[user@localhost html]$ wget http://osticket.com/sites/default/files/download/osTicket-v1.9.12.zip
[user@localhost html]$ unzip ./osTicket-v1.9.12.zip
[user@localhost html]$ sudo chown apache:apache -R ./upload/*
[user@localhost html]$ sudo cp upload/include/ost-sampleconfig.php upload/include/ost-config.php
[user@localhost html]$ sudo chmod 0666 upload/include/ost-config.php
[user@localhost html]$ cd upload/include/i18n/
[user@localhost html]$ wget http://osticket.com/sites/default/files/download/lang/pt_BR.phar
```
Agora basta terminar a instalação pelo browser.
