# Instalação do Client do Bareos(bareos-fd) 15.2.2

## Download

Adicione o repositório correto para a distribuição que
você está utilizando. Aqui temos exemplos para o
**CentOS 7** e também para o **CentOS 6**.

### CentOS 6
```sh
$ sudo wget -O /etc/yum.repos.d/bareos.repo http://download.bareos.org/bareos/release/latest/CentOS_6/bareos.repo 
```

### Centos 7
```sh
$ sudo wget -O /etc/yum.repos.d/bareos.repo http://download.bareos.org/bareos/release/latest/CentOS_7/bareos.repo 
```
Agora o Bareos está disponível para instalar via o yum
```sh
$ sudo yum install bareos-fd
```

## Configuração

O processo de configuração de um cliente é bem mais simples
do que o processo necessário para a configuração de um
servidor. Basicamente o que precisa ser feito são 3 coisas:

-Liberação de portas no firewall
-Configuração do *bareos-fd.conf* no cliente
-Configuração do *bareos-dir.conf* no servidor

### Firewall

O processo de configuração do firewall é específico para
cada sistema. A seguir temos exemplos para o **iptables**
e também para o **firewalld**.


#### **iptables**

```sh
$ sudo iptables -I INPUT -p tcp -m tcp --dport 9102 -j ACCEPT
$ sudo iptables -I OUTPUT -p tcp -m tcp --dport 9102 -j ACCEPT
$ sudo /sbin/service iptables save
```

#### **firewalld**


```sh
$ sudo firewall-cmd --zone=public --add-port=9102/tcp
$ sudo firewall-cmd --zone=public --add-port=9102/tcp --permanent
```

### *bareos-fd.conf*

No arquivo de configuração do cliente, que fica localizado
no diretório */etc/bareos/*, temos 2 seções importantes
para serem configuradas. A primeira é a seção onde é
realizada as configurações do próprio cliente.

```apache
FileDaemon { # definition of myself
    Name = client-fd
    Maximum Concurrent Jobs = 20
}
```

A próxima etapa é listar todos os *director* que podem
se conetar neste cliente. Neste exemplo temos apenas 1
*director*, que está em outro servidor.

```apache
# List Directors who are permitted to contact this File daemon
Director {
    Name = bareos-dir
    Password = "senha/aleatoria/e/bem/segura"
}
```
### Reiniciando o serviço

Neste momento já temos todas as configurações necessárias
para o funcionamento correto do cliente, entretanto 
é necessário que se reinicie o serviço *bareos-fd* para
que as novas configurações sejam carregadas corretamente.

#### CentOS 6

```sh
$ sudo /etc/init.d/bareos-fd restart
Stopping Bareos File services: [  OK  ]
Starting Bareos File services: [  OK  ]
$ sudo /etc/init.d/bareos-fd status
bareos-fd (pid 1471) is running...
```
#### CentOS 7

```sh
$ sudo systemctl restart bareos-fd
$ sudo systemctl status bareos-fd
● bareos-fd.service - Bareos File Daemon service
   Loaded: loaded (/usr/lib/systemd/system/bareos-fd.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2016-08-25 11:01:27 BRT; 11s ago
     Docs: man:bareos-fd(8)
  Process: 2400 ExecStart=/usr/sbin/bareos-fd -c /etc/bareos/bareos-fd.conf (code=exited, status=0/SUCCESS)
 Main PID: 2401 (bareos-fd)
   CGroup: /system.slice/bareos-fd.service
           └─2401 /usr/sbin/bareos-fd -c /etc/bareos/bareos-fd.conf
```

### *bareos-dir.conf*


