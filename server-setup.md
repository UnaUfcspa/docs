Setup inicial de um servidor
============================

Este documento é uma coleção de dicas para tornar mais facil e padronizada a tarefa de entregar um
servidor novo para o ambiente de produção.
Tudo que está descrito aqui é baseado no **CentOS 7**, mas apesar disto as ideias apresentas são relevantes 
para a maior parte das distribuições atuais.

Indice
------
  - [Adicionar usuários](#adicionar-usuários)
  - [sshd](#sshd)
  - [Atualização do sistema](#atualização-do-sistema)
  - [SELinux](#selinux)
  - [Horário](#configurar-o-horário)
  - [Firefall](#firewall)
  - [Hostname](#hostname)

## Adicionar usuários

```sh
[root@localhost /]# useradd user -g wheel -p<senha>
```

## sshd

Dependendo da configuração do *sshd* é possível que você não consiga realizar login via ssh utilizando o usuário recém criado.
Neste caso, em geral, basta adicionar o novo usuário no arquivo de configuração do *sshd* para que o problema seja resolvido.
A localização completa do arquivo é */etc/ssh/sshd_config*.
Este é um bom momento para alterar a porta do *sshd*, caso seja de seu interesse.

```apache
Port 1234
AllowUsers user
```
Não esqueça de reiniciar o *sshd* depois de realizar as alterações.

```sh
[root@localhost /]$ systemctl restart sshd
[root@localhost /]$ systemctl status sshd
● sshd.service - OpenSSH server daemon
   Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2016-08-02 11:01:19 BRT; 1 weeks 0 days ago
     Docs: man:sshd(8)
           man:sshd_config(5)
 Main PID: 1385 (sshd)
   CGroup: /system.slice/sshd.service
           └─1385 /usr/sbin/sshd -D

Aug 02 11:01:19 localhost.localdomain systemd[1]: Started OpenSSH server daemon.
Aug 02 11:01:19 localhost.localdomain systemd[1]: Starting OpenSSH server daemon...
Aug 02 11:01:19 localhost.localdomain sshd[1385]: Server listening on 0.0.0.0 port 1234.
Aug 02 11:01:19 localhost.localdomain sshd[1385]: Server listening on :: port 1234.
```

> **Atenção**: Ao mudar as configurações do *sshd* certifique-se de que você consegue estabelecer
> uma nova conexão antes de fechar a atual.

## Atualização do sistema

```sh
[user@localhost ~]$ sudo yum update
```
## SELinux

Verificando que o *SELinux* está ativo e no modo *enforcing* não há nada mais para fazer.
```sh
[user@localhost docs]$ sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      28
```
Se o SELinux estiver no modo *Permissive*, ele pode ser ativado através do comando *setenforce*

```sh
[user@localhost docs]$ sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   permissive
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      28
[user@localhost ~]$ sudo setenforce 1
```
Já se ele estiver desativado será necessário alterar o arquivo */etc/selinux/config*.

```apache
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted 
```
Ao mudar a configuração do *SELinux* de *disabled* para *permissive* ou *enforcing* é necessário que o sistema seja 
reiniciado para que ele possa criar as labels necessárias.

A melhor estratégia para habilitar o *SELinux* em um servidor de produção é primeiro coloca-lo no modo *permissive*
e resolver os problemas de permissões e somente depois que estiver tudo resolvido mudar para o modo *enforcing*.

## Configurar o horário

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
## Firewall

Verifique se o firewall está devidamente instalado e configurado no servidor.

```sh
[user@localhost ~]$ sudo firewall-cmd --permanent --list-all
public (default)
  interfaces:
  sources:
  services: dhcpv6-client ssh
  ports: 
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
[user@localhost ~]$ sudo systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled;
vendor preset: enabled)
   Active: active (running) since Tue 2016-08-02 11:01:11 BRT; 1 weeks
0 days ago
 Main PID: 742 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─742 /usr/bin/python -Es /usr/sbin/firewalld --nofork --...

Aug 02 11:01:09 localhost.localdomain systemd[1]: Starting firewalld...
Aug 02 11:01:11 localhost.localdomain systemd[1]: Started firewalld ...
Hint: Some lines were ellipsized, use -l to show in full.
```
## Hostname

A configuração do hostname do servidor pode ser realizada através dos arquivos */etc/sysconfig/network*
e */etc/hosts*.

```apache
# Arquivo /etc/sysconfig/network
# Created by anaconda
HOSTNAME=hostname.com.br
```

```apache
# Arquivo /etc/hosts
127.0.0.1   hostname hostname.com.br 
::1         hostname hostname.com.br
```

Depois de realizar as configurações basta apenas reiniciar o *network*.
```sh
[user@localhost ~]$ sudo systemctl restart network
[user@localhost ~]$ sudo systemctl status network
● network.service - LSB: Bring up/down networking
   Loaded: loaded (/etc/rc.d/init.d/network)
   Active: active (exited) since Tue 2016-08-02 11:01:19 BRT; 1 weeks 1 days ago
     Docs: man:systemd-sysv-generator(8)
```
## Log rotate
## fail2ban
