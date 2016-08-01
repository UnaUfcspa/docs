# Instalação do Bareos 15.2.2

Este conteúdo é baseado no manual oficial do Bareos, disponível [aqui][doc].

## Setup

Instalar e habilitar o MariaDB
```sh
# yum install mariadb-server
# systemctl enable mariadb-server
# systemctl start mariadb-server
```

Habilitar o repositório de pacotes extras do Centos
```sh
# yum install epel-release
```

## Download

Adicionar o repositório no servidor
```sh
# wget -O /etc/yum.repos.d/bareos.repo http://download.bareos.org/bareos/release/latest/CentOS_7/bareos.repo 
```
Agora o Bareos está disponível para instalar via o yum
```sh
# yum install bareos
```
Usaremos o MariaDB para armazenar as informações do Bareos
```sh
# yum install bareos-database-mysql
```

## Configuração

Executar estes scrips para inicializar o banco de dados
```sh
/usr/lib/bareos/scripts/create_bareos_database 
/usr/lib/bareos/scripts/make_bareos_tables 
/usr/lib/bareos/scripts/grant_bareos_privileges
```

Iniciar e habilitar o serviço do Bareos
```sh
# systemctl start bareos-*
# systemctl enable bareos-*
```

## Bareos-webui

```sh
# yum install bareos-webui
```

## Selinux
A interface web precisa receber permissão para funcionar corretamente
```sh
# setsebool -P httpd_can_network_connect on
```



   [doc]: <http://doc.bareos.org/master/html/bareos-manual-main-reference.html#x1-250002> 

