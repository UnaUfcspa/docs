# Instalação do Bareos 15.2.2

Este conteúdo é baseado no manual oficial do Bareos, disponível [aqui][doc].

## Setup

Instalar e habilitar o MariaDB
```sh
$ sudo yum install mariadb-server
$ sudo systemctl enable mariadb-server
$ sudo systemctl start mariadb-server
```

Habilitar o repositório de pacotes extras do Centos
```sh
$ sudo yum install epel-release
```

## Download

Adicionar o repositório no servidor
```sh
$ sudo wget -O /etc/yum.repos.d/bareos.repo http://download.bareos.org/bareos/release/latest/CentOS_7/bareos.repo 
```
Agora o Bareos está disponível para instalar via o yum
```sh
$ sudo yum install bareos
```
Usaremos o MariaDB para armazenar as informações do Bareos
```sh
$ sudo yum install bareos-database-mysql
```

## Configuração inicial

Executar estes scripts para inicializar o banco de dados
```sh
/usr/lib/bareos/scripts/create_bareos_database 
/usr/lib/bareos/scripts/make_bareos_tables 
/usr/lib/bareos/scripts/grant_bareos_privileges
```

Iniciar e habilitar o serviço do Bareos
```sh
$ sudo systemctl start bareos-*
$ sudo systemctl enable bareos-*
```

## Bareos-webui

```sh
$ sudo yum install bareos-webui
```

## Selinux
A interface web precisa receber permissão para funcionar corretamente
```sh
$ sudo setsebool -P httpd_can_network_connect on
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
O nome e a senha, definidos nesta seção, devem ser os mesmos valores utilizados no arquivo *bconsole.conf*.

#### Definição de clientes
Para definir um cliente é necessário especificar 3 parâmetros: Name, Address e Password.

```apache
Client {
    Name = cliente-fd
    Address = 192.168.1.1
    Password = "senhadocliente"
}
```
A **senha** e o **nome** devem ser iguais aos valores que foram definidos no arquivo *bareos-fd.conf* do cliente.

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
Aqui são definidos os arquivos que devem salvos. Esta configuração é independente dos clientes. É na definição de *jobs* que serão relacionados *filesets*, *agendamentos* e clientes.

É em *Include* que são definidos os diretórios que devem ser salvos durante o backup. E, de maneira similiar em *Exclude* são definidos os que não serão salvos.
Note que os diretórios são definidos **sem** o '/' no final.
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

### File daemon
A configuração do bareos-fd fica no arquivo /etc/bareos/bareos-fd.conf
```apache
FileDaemon {
    Name = cliente-fd
    Maximun Concurrent Jobs = 20
}
```

```apache
Director {
    Name = bareos-dir
    Password = "senhadocliente"
}
```
Esta senha deve ser a mesma definida no arquivo *bareos-dir.conf*.

### Storage daemon
A configuração do bareos-sd fica no arquivo /etc/bareos/bareos-sd.conf
```apache
Storage {
    Name = cliente-sd
    Maximum Concurrent Jobs = 20
}
```

```apache
Director {
    Name = bareos-dir
    Password = "senhabareos"
}
```

```apache
Device {
    Name = FileStorage
    Media Type = File
    Archive Device = /var/lib/bareos/storage
    LabelMedia = yes;
    Random Access = yes;
    AutomaticMount = yes;
    RemovableMedia = no;
    AlwaysOpen = no;
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

