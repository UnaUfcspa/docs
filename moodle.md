# Instalação do Moodle 3.1

Este guia foi criado com base na documentação oficial, diponível [aqui](https://docs.moodle.org/31/en/Installing_Moodle).

## Requisitos de software

O principal problema que encontramos ao realizar a instalação da versão 3.1 do Moodle foi
relacionado à versão necessária do **mysql**. A partir do Moodle 2.7 a versão mínima do **mysql**
é a 5.5.31, entretanto no CentOS 6 a última versão disponível *oficialmente* é a 5.1. Isso pode ser
um problema bem complicado de resolver se você estiver tentando realizar o upgrade de uma versão
mais antiga do Moodle para a versão mais nova.

## Instalando os serviços necessários

Para o funcionamento do Moodle são necessários 3 serviços básicos no servidor um servidor de web,
um sgbd e o php. Começaremos a instalação pelo servidor web.

### Apache

A nossa escolha foi o **Apache** mas nada impede que outro servidor seja utilizado.

```sh
[user@localhost ~]$ sudo yum install httpd
[user@localhost ~]$ sudo systemctl start httpd
[user@localhost ~]$ sudo systemctl enable httpd
[user@localhost ~]$ sudo systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running)
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 31433 (httpd)
   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /system.slice/httpd.service
           ├─10989 /usr/sbin/httpd -DFOREGROUND
           ├─29040 /usr/sbin/httpd -DFOREGROUND
           ├─29195 /usr/sbin/httpd -DFOREGROUND
           ├─29844 /usr/sbin/httpd -DFOREGROUND
           ├─29868 /usr/sbin/httpd -DFOREGROUND
           ├─29965 /usr/sbin/httpd -DFOREGROUND
           ├─30656 /usr/sbin/httpd -DFOREGROUND
           ├─30683 /usr/sbin/httpd -DFOREGROUND
           ├─30799 /usr/sbin/httpd -DFOREGROUND
           ├─30847 /usr/sbin/httpd -DFOREGROUND
           └─31433 /usr/sbin/httpd -DFOREGROUND

```

### MariaDB

```sh
[user@localhost ~]$ sudo yum install mariadb-server
[user@localhost ~]$ sudo systemctl start mariadb-server
[user@localhost ~]$ sudo systemctl enable mariadb-server
● mariadb.service - MariaDB database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
   Active: active (running)
 Main PID: 15127 (mysqld_safe)
   CGroup: /system.slice/mariadb.service
           └─15127 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
```

### PHP
```sh
sudo yum install php php-iconv php-mbstring php-curl php-openssl php-tokenizer
sudo yum install php-xmlpc php-soap php-ctype php-zip php-gd php-simplexml
sudo yum install php-spl php-pcre php-dom php-xml php-intl php-json php-ldap
sudo yum install php-pecl-apc php-xmlrpc
sudo yum install php-mysql phpmyadmin
```

