# Restaurando backups

Este documento contém as informações necessárias para que seja realizado um restore
utilizando o **BAREOS**.

O restore pode ser feito tanto pelo console como por sua interface web. Aqui abordaremos
a segunda alternativa porque ela é mais simples.

Na aba *restore* existem várias opções que podem tornar o trabalho de restore bastante
flexível.

## Client

Esta opção se refere ao cliente de "origem". Ao realizar uma alteração nesta opção serão
apresentados novas opções para os outros valores.

## Backup jobs

Aqui devemos selecionar qual o backup que deverá ser restaurado. O número que aparece na frente
do nome é o *id* do job, depois temos a data da realização do backup. O próximo valor é o nome
definido na configuração do job e por fim temos o tipo do backup(Incremental, Diferential ou Full).

## Merge all client filesets?

O backup de um cliente pode ser separado em vários jobs para permitir um melhor gerenciamento das
configurações. Esta opção permite que em um único restore se faça a restauração de arquivos contidos
em todos os diferentes conjuntos de arquivos definidos para aquele cliente. Em geral pode ser
utilizado *yes* sem problema nenhum.

## Merge all related jobs to last full backup of selected backup job?

As operações de backup podem ser realizadas em vários niveis, com os principais sendo *Incremental*
e *Full*. A diferença entre estes niveis é exatamente o que os nomes sugerem. O segundo realiza um
backup completo e o primeiro só salva os arquivos que sofreram alterações depois do último backup
completo. Assim essa opção permite que em apenas um restore se recupere arquivos salvos em vários
backups incrementais. Em geral pode ser utilizado *yes* sem problema nenhum.

## Restore to (another) client

Esse sistema de backup é flexível o suficiente que permite que o backup realizado em um cliente seja
restaurado em outro cliente. Essa é uma opção interessante para recuperação de desastres, permitindo
que seja facilmente duplicado um ambiente completo. Uma limitação é que só pode ser realizado o
restore em clientes já configurados neste *director*.

## Restore job

Esta opção pode ser ignorada sem problemas. Ela existe, basicamente, por questões históricas.

## Replace files on client?

Aqui se define qual o comportamento que será adotado quando for encontrado um conflito de arquivos.

## Restore location on client

Este é o diretório onde os arquivos serão restaurados no cliente.

## Select files to be restored

Nesta área será apresntada um árvore de diretórios contendo todos os arquivos que podem ser
restaurados a partir das opções selecionadas.

> **Nota**: Cada vez que se realiza a alteração de uma opção **todos** os outros valores abaixo
> daquele alterado são resetados para seus valores iniciais. Por isso é interessante que o
> formulário seja preenchido na ordem apresentada.

Depois que tudo estiver corretamente preenchido é só clicar no botão **Restore** que está no
final da página que a restauração irá começar.
