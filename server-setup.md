# Configuração de usuários


```sh
[root@localhost /]# useradd user -g wheel -p<senha>
```

Adicionar o novo usuário no arquivo de configuração do *sshd*. A localização completa do arquivo é */etc/ssh/sshd_config*

```apache
AllowUsers usuarionovo
```

# Atualização do sistema

```sh
[user@localhost ~]$ sudo yum update
```
# Verificando o status do SELinux

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
Fazer o que se não estiver habilitado??!!

# Configurar o horário

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
# Firewall

Verifique se o firewall está devidamente instalado e configurado no servidor.

```sh
[user@localhost ~]$ sudo firewall-cmd --permanent --list-all
public (default)
  interfaces:
  sources:
  services: dhcpv6-client ssh
  ports: 9103/tcp 9101/tcp
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
[user@localhost docs]$ sudo systemctl status firewalld
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
# Hostname

# Log rotate
# sshd
# fail2ban
