Este documento contém os passos necessários para configurar uma unidade de disco extra.

## Identificando o disco

O primeiro passo é verificar qual o endereço do novo disco. Existem, pelo menos, duas maneiras de
realizar a identificação do disco, uma utilizando o *lsblk* e outra com o *parted*.

```sh
[user@localhost ~]$ lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0              11:0    1 1024M  0 rom
xvda            202:0    0  100G  0 disk
├─xvda1         202:1    0  500M  0 part /boot
└─xvda2         202:2    0 99.5G  0 part
  ├─centos-root 253:0    0   50G  0 lvm  /
  ├─centos-swap 253:1    0  7.6G  0 lvm  [SWAP]
  └─centos-home 253:2    0 41.8G  0 lvm  /home
xvdb            202:16   0  200G  0 disk
```

Neste caso temos um disco com o nome **xvdb** com o tamanho aproximado de 200 GB.

Outra forma de identificar o disco é utilizando o *parted*, porém a saída dele é uma pouco mais
complicada de interpretar. Nela você deve procurar por uma entrada identificada como *Error*, como
a apresentada a seguir:

```sh
[user@localhost ~]$ sudo parted -l
...
Error: /dev/xvdb: unrecognised disk label
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvdb: 215GB
Sector size (logical/physical): 512B/512B
Partition Table: unknown
Disk Flags:
```
Uma alternativa seria utilizar o *grep* em conjunto com o *parted* para não precisar ficar
procurando o resultado desejado manualmente mas ao utilizar essa estratégia as informações
adicionais não serão exibidas, isso pode ou não ser um problema.

```sh
[user@localhost ~]$ sudo parted -l | grep Error
Error: /dev/xvdb: unrecognised disk label
```

## Criando uma partição

Agora que já realizamos a identificação do disco que deve ser preparado podemos partir para o
próximo passo, que é a criação da partição neste disco.

Atualmente existem dois padrões de tabelas de partição o **MBR** e o **GPT**. Sendo que o primeiro
já está entrando em declínio, enquanto o segundo é um padrão mais novo e com várias vantagens
sobre o primeiro.

As principais vantagens do **GPT** em relação ao **MBR** são a possibilidade de criar uma partição
com tamanho maior do que 2 TB e a possibilidade de criar mais do que 4 partições em um mesmo disco.

### Tabela de partição

Para criar uma nova tabela de partição **GPT** utilize o seguinte comando:

```sh
[user@localhost ~]$ sudo parted /dev/xvdb mklabel gpt
Information: You may need to update /etc/fstab.
```
### Nova partição

Nós usamos o formato **xfs** para as nossas partições, porque este é um formato mais robusto
e próprio para ambientes que precisam de maior estabilidade e performance. Apesar disto, se você
prefere utilizar outro formato, como o ext4, não há nenhum problema.

Esta operação pode ser realizada de duas maneiras, utilizando o parted de maneira interativa ou de
maneira direta. Escolhemos a maneira direta porque tudo pode ser feito com apenas um comando.

```sh
[user@localhost ~]$ sudo parted -a opt /dev/xvdb mkpart primary xfs 0% 100%
Information: You may need to update /etc/fstab.
```

> Note que criamos uma partição **primária** que ocupa todo nosso disco. Além disso para especificar
> o tamanho da partição nós utilizamos percentuais. A vantagem de fazer desta forma é a partição
> criada será alinhada com os setores físicos de forma ótima, aumentando a performance do disco.

Neste momento a nova partição já está criada, e isso pode ser verificado utilizando o *lsblk*

```sh
[user@localhost ~]$ lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0              11:0    1 1024M  0 rom
xvda            202:0    0  100G  0 disk
├─xvda1         202:1    0  500M  0 part /boot
└─xvda2         202:2    0 99.5G  0 part
  ├─centos-root 253:0    0   50G  0 lvm  /
  ├─centos-swap 253:1    0  7.6G  0 lvm  [SWAP]
  └─centos-home 253:2    0 41.8G  0 lvm  /home
xvdb            202:16   0  200G  0 disk
└─xvdb1         202:17   0  200G  0 part
```

Depois de criada a partição não possui um formato. Ele pode ser definido com o seguinte comando:

```sh
[user@localhost ~]$ sudo mkfs.xfs -L storage /dev/xvdb1
meta-data=/dev/xvdb1             isize=256    agcount=4, agsize=13107072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=0        finobt=0
data     =                       bsize=4096   blocks=52428288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=0
log      =internal log           bsize=4096   blocks=25599, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

Para verificar se a partição foi formatada de maneira correta podemos usar novamente o *lsblk*

```sh
[user@localhost ~]$ lsblk --fs
NAME            FSTYPE      LABEL   UUID                                   MOUNTPOINT
sr0
xvda
├─xvda1         xfs                 03e45f18-1241-43af-a26e-30d85be0062f   /boot
└─xvda2         LVM2_member         6UjtDa-EBK8-aD4s-ehAu-XgFf-qaLT-VwyInw
  ├─centos-root xfs                 208a4124-e3b0-4bd7-93f2-678612e9de2c   /
  ├─centos-swap swap                a69b10aa-8bce-43a7-81bb-9a69bd577a89   [SWAP]
  └─centos-home xfs                 9138b892-5f8a-43ea-8b45-ab07de522c4e   /home
xvdb
└─xvdb1         xfs         storage e7d08048-58ad-4202-b8ad-1ab46f3942b0
```

## Montando a partição

Esta nova partição será montada na pasta */media/storage*, para isso vamos criar uma nova entrada no
arquivo */etc/fstab* para refletir essa mudança.

```apache
LABEL=storage /media/storage                   xfs     defaults        0 0
```

Feito isto agora esta partição sempre será montada automaticamente quando o servidor for reiniciado.
Se o servidor não puder ser reiniciado neste momento a partição pode ser montada com o comando *mount*;

Depois de montada a partição pode ser verificada utilizando o comando *df*
```sh
[user@localhost ~]$ df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root   50G  1.3G   49G   3% /
devtmpfs                 3.9G     0  3.9G   0% /dev
tmpfs                    3.7G     0  3.7G   0% /dev/shm
tmpfs                    3.7G  8.4M  3.7G   1% /run
tmpfs                    3.7G     0  3.7G   0% /sys/fs/cgroup
/dev/xvdb1               200G   33M  200G   1% /media/storage
/dev/mapper/centos-home   42G   33M   42G   1% /home
/dev/xvda1               497M  210M  287M  43% /boot
tmpfs                    757M     0  757M   0% /run/user/1002
```
