# Instalação do Bareos Director, versão 15.2.2

Este conteúdo é baseado no manual oficial do Bareos, disponível [aqui][doc].

## Esclarecimentos

O Bareos é um sistema de backup modular, o que permite que sua instalação
seja customizada para as suas necessidades.

Em nosso caso escolhemos hospedar o SGBD, o *director*, o *storage* e a
interface web no mesmo servidor. Este servidor é identificado pelo nome
*bareos-server*. Durante a construção deste documento nós procuramos
tomar cuidado para sempre incluir o nome do servidor onde os comandos
devem ser executados. Essa informação está explícita no *prompt primário*
do BASH que é apresentado junto com os comandos que devem ser executados.

Por fim, este documento trata apenas da instalação do bareos-director
no CentOS 7. A instalação do storage, [client][cli], configurações de backup
e o [procedimento de restore][restore] são apresentados em outros documentos.

## Requisitos

O principal requisito de software para Bareos é o SGBD que será utilizado para
guardar as informações referentes aos backups realizados. As opções oferecidas
são PostgreSQL, MySQL(MariaDB) e Sqlite. Apesar de ser uma opção suportada, o
Sqlite como SGDB não é recomendado para utilização em ambiente de produção. Ele
é mais adequado para ambientes de teste.

Além disto devemos considerar a necessidade de um servidor web para que possamos
utilizar a interface web do software.

## Setup

Em nosso caso escolhemos utilizar a combinação MariaDB + Apache como base para
nosso sistema de backup. Essa escolha não deve ter um impacto muito grande em
relação à complexidade da instalação do sistema.

Antes de continuar devemos mencionar que as instruções apresentadas a seguir
são específicas para o CentOS 7.2 e podem existir grandes variações para
realizar o mesmo processo em outras versões deste sistema operacional.

### SGBD

O primeiro passo para a instalação do Bareos é instalar e habilitar o SGBD escolhido,
que em nosso caso é o MariaDB. Ele se encontra disponível nos repositórios oficiais
e pode ser instalado através do **yum**.

```sh
[user@bareos-server ~]$ sudo yum install mariadb-server
```

Depois de instalado o serviço precisa ser habilitado, para que no futuro inicie junto
com o sistema, e iniciado. No CentOS 7 podemos utilizar o **systemctl** para esta
finalidade.

```sh
[user@bareos-server ~]$ sudo systemctl enable mariadb
[user@bareos-server ~]$ sudo systemctl start mariadb
```

### Web Server

Como iremos instalar a interface web neste servidor precisaremos instalar um servidor
web. Como já comentamos, a nossa escolha é o Apache que tembém pode ser instalado
diretamente pelo **yum**.

```sh
[user@bareos-server ~]$ sudo yum install httpd
```

Da mesma forma que o SGBD, o servidor web também precisa ser habilitado e iniciado
logo após a sua instalação.

```sh
[user@bareos-server ~]$ sudo systemctl start httpd
[user@bareos-server ~]$ sudo systemctl enable httpd
```

O firewall padrão do CentOS 7 é o **firewalld** que pode ser gerenciado pelo
**firewall-cmd**, sua utilização é muito mais simples do que o **iptables**, por
exemplo. Para liberar a porta 80/tcp de forma definitiva em nosso servidor basta
executar os seguintes comandos:

```sh
[user@bareos-server ~]$ sudo firewall-cmd --zone=public --add-port=http/tcp
[user@bareos-server ~]$ sudo firewall-cmd --zone=public --add-port=http/tcp --permanent
```

Para que a interface web funcione corretamente, além da liberação no firewall, é necessário
setar uma permissão específica do **SELinux**.

```sh
[user@bareos-server ~]$ sudo setsebool -P httpd_can_network_connect on
```

> Colocar o **SELinux** no modo *permissivo* **NÃO** é uma boa ideia para ambientes de produção
> mas pode evitar muita dor de cabeça em ambientes de teste.

Habilitar o repositório de pacotes extras do Centos
```sh
[user@bareos-server ~]$ sudo yum install epel-release
```
## Download

```sh
[user@bareos-server ~]$ sudo yum install wget
```

Adicionar o repositório no servidor
```sh
[user@bareos-server ~]$ sudo wget -O /etc/yum.repos.d/bareos.repo http://download.bareos.org/bareos/release/latest/CentOS_7/bareos.repo 
```
Agora o Bareos está disponível para instalar via o yum
```sh
[user@bareos-server ~]$ sudo yum install bareos
```
Usaremos o MariaDB para armazenar as informações do Bareos
```sh
[user@bareos-server ~]$ sudo yum install bareos-database-mysql
```

## Configuração inicial

Executar estes scripts para inicializar o banco de dados
```sh
[user@bareos-server ~]$ sudo /usr/lib/bareos/scripts/create_bareos_database
[user@bareos-server ~]$ sudo /usr/lib/bareos/scripts/make_bareos_tables
[user@bareos-server ~]$ sudo /usr/lib/bareos/scripts/grant_bareos_privileges
```

Iniciar e habilitar o serviço do Bareos
```sh
[user@bareos-server ~]$ sudo systemctl start bareos-dir
[user@bareos-server ~]$ sudo systemctl start bareos-sd
[user@bareos-server ~]$ sudo systemctl enable bareos-dir
[user@bareos-server ~]$ sudo systemctl enable bareos-sd
```

## Bareos-webui


```sh
[user@bareos-server ~]$ sudo yum install bareos-webui
```


## Arquivos de configuração

### Director
A configuração do bareos-dir fica no arquivo /etc/bareos/bareos-dir.conf.
Aqui ficam as principais configurações do bareos, como por exemplo definições de:

    * clientes
    * storages
    * jobs
    * conjuntos de arquivos
    * agendamentos de backups

#### Definição do director

```apache
Director {
    Name = bareos-dir
    QueryFile = "/usr/lib/bareos/scripts/query.sql"
    Maximum Concurrent Jobs = 10
    Password = "senhadodirector"
    Messages = Daemon
    Auditing = yes
}
```
O nome e a senha, definidos nesta seção, devem ser os mesmos valores utilizados
no arquivo *bconsole.conf*.

#### Definição de clientes
Para definir um cliente é necessário especificar 3 parâmetros: Name, Address e Password.

```apache
Client {
    Name = cliente-fd
    Address = 192.168.1.1
    Password = "senhadocliente"
}
```
A **senha** e o **nome** devem ser iguais aos valores que foram definidos no
arquivo *bareos-fd.conf* do cliente.

#### Definição de storages
```apache
Storage {
    Name = File
    Address = 192.168.1.1
    Password = "senhadocliente"
    Device = FileStorage
    Media Type = File
}
```

#### Definição de filesets
Aqui são definidos os arquivos que devem salvos. Esta configuração é
independente dos clientes. É na definição de *jobs* que serão relacionados
*filesets*, *agendamentos* e clientes.

É em *Include* que são definidos os diretórios que devem ser salvos durante o
backup. E, de maneira similiar em *Exclude* são definidos os que não serão
salvos.  Note que os diretórios são definidos **sem** o '/' no final.
```apache
FileSet {
    Name = "NomedoConjunto"
    Include {
        Options {
            Signature = MD5
        }
    File = /var/www/html
    }
    Exclude {
        File = /var/lib/bareos
        File = /proc
        File = /tmp
    }
}
```

#### Definição de jobs

```apache
JobDefs {
    Name = "DefaultJob"
    Type = Backup
    Level = Incremental
    Storage = File
    Messages = Standard
    Pool = Incremental
    Priority = 10
    Write Bootstrap = "/var/lib/bareos/%c.bsr"
    Full Backup Pool = Full
    Differential Backup Pool = Differential
    Incremental Backup Pool = Incremental
}
```

```apache
Job {
    Name = "Serv2 OSTicket Backup"
    Client = serv2-fd
    JobDefs = "DefaultJob"
    Schedule = "Diario"
    FileSet = "OSTicket"
    ClientRunBeforeJob = "/var/spool/bareos/mariadump.sh"
    ClientRunAfterJob = "/var/spool/bareos/mariaclean.sh"
}
```

### bconsole

```apache
Director {
    Name = localhost-dir
    DIRport = 9101
    address = localhost
    Password = "senhadodirector"
}
```

## bareos-webui

No arquivo *bareos-dir.conf* é necessário acrescentar as definições da interface web para que a interface funcione de maneira correta.

```apache
@/etc/bareos/bareos-dir.d/webui-consoles.conf
@/etc/bareos/bareos-dir.d/webui-profiles.conf
```
### webui-consoles

```apache
Console {
    Name = user1
    Password = "senhaweb"
    Profile = webui
}
```

### webui-profiles
```apache
Profile {
    Name = webui
    CommandACL = status, messages, show, version, run, rerun, cancel, .api, .bvfs_*, list, llist, use, .jobs, .filesets, .clients
    Job ACL = *all*
    Schedule ACL = *all*
    Catalog ACL = *all*
    Pool ACL = *all*
    Storage ACL = *all*
    Client ACL = *all*
    FileSet ACL = *all*
    Where ACL = *all*
}
```


   [doc]: <http://doc.bareos.org/master/html/bareos-manual-main-reference.html#x1-250002>
   [cli]: <https://github.com/UnaUfcspa/docs/blob/master/bareos/client-install.md>
   [restore]: <https://github.com/UnaUfcspa/docs/blob/master/bareos/restore.md>


