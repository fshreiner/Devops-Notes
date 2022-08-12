Conhecendo o Shell Provisioner
Como vimos, o Vagrant exerce a função de configurar nossa máquina virtual a partir de um arquivo de texto. No entanto, há uma série de características detalhadas de software, ambiente e usuários que o programa não é capaz de abarcar, como instalar _endings_ como o MySQL por exemplo, adicionar automaticamente uma chave extra gerada por nós, entre outras.

Para isso, vamos à documentação para acessar a parte de "Provisioning", onde encontramos informações sobre como preparar a máquina para uso de forma automatizada e programável. O primeiro item que veremos e que faz parte desta sessão é o "Shell Provisioner".

No Linux, o **Shell** - ou também o Batch - é o local onde se executa os comandos, e através do Vagrant podemos definir quais serão estes.

Neste [endereço](http://www.vagrantup.com/docs/provisioning/shell.html) temos a configuração apresentada em "Inline Scripts" que copiamos e colamos no arquivo "Vagrantfile" no editor de texto.

```
Vagrant.configure("2") do |config|
    config.vm.provision "shell",
        inline: "echo hello, World"
end
```

Após salvar, vá ao terminal com a máquina virtual rodando verificada com `vagrant status` para recarregar com `vagrant reload`. Apesar de não haver alterações, aparece uma linha escrita "Machine already provisioned", indicando que esta já foi provisionada e que devemos inserir o comando `vagrant provision`.

Feito isso, os provisionamentos foram executados na MV e assim vemos a saída. Para visualizarmos, volte ao editor de texto e refaça a ultima linha alterando para:

```
    config.vm.provision "shell", inline: "echo Hello, World >> hello.txt"
end
```

Execute novamente o último comando no terminal para rodar apenas os provisionadores. Assim, a linha que apresenta `default: Hello World` desaparece.

Ao nos conectarmos com `vagrant ssh`, limparmos a tela, executarmos `ls` para visualizarmos `hello.txt id_bionic.pub` e por fim escrevermos `cat hello.txt` observamos que este comando foi executado dentro da máquina virtual que criou o arquivo a partir das últimas alterações em "Vagrantfile".

Desta forma, podemos executar comandos **Batch-Shell** através dessas configurações, nos possibilitando mais recursos sofisticados como a instalação do Nginx ou MySQL por exemplo.

No próximo passo, veremos como fazer isso.



Synced Folder e mais shell
Continuaremos a falar sobre _provisioning_ para que a máquina virtual seja preparada e funcione com recursos mais sofisticados.

Com as configurações anteriores, mesmo após destruição e recriação terminando por inserir o comando `vagrant up`no terminal, o programa já executa o provisionador Shell automaticamente.

Repare também que há dois novos aspectos apresentados: `default: Mounting shared folders...` e `default: /vagrant => D:/ambiente_dev/bionic`. Nos passos anteriores quando vimos sobre a chave pública, aprendemos que junto com o provisioning há a necessidade de usar a pasta que contém a ligação entre o host do Windows e a máquina virtual. Ou seja, o diretório "bionic" só é visível no Linux a partir do "vagrant".

Este processo é importante para não só instalar um Nginx, mas também configurá-lo. Logo, a partir do Windows podemos por exemplo copiar uma configuração sobre hosts virtuais para o servidor escolhido e reiniciar, transferindo arquivos do sistema operacional para a máquina virtual com _shared folder_.

A chave pública `id_bionic.pub` foi adicionada na pasta "vagrant" anteriormente. Usaremos este exemplo para demonstrar o processo; quando escrevemos `cat .ssh/authorized_keys` percebemos que só há a chave padrão do próprio Vagrant, e queremos que a chave pública que criamos esteja presente automaticamente quando executamos o programa pela primeira vez.

Para isso, fechamos com `exit` e criamos uma nova pasta dentro de "bionic" chamada "configs", que receberá `id_bionic.pub` e armazenará os demais arquivos de configuração para deixar mais clara sua função.

Agora montaremos este diretório para a máquina virtual. Indo para a parte "Synced Folders" na documentação do Vagrant, obtemos a seguinte sintaxe de referência que será alterada para adicionarmos ao "Vagrantfile":

```
    config.vm.synced_folder "./configs", "/configs"
end
```

Salva a adição no editor de texto, voltamos ao terminal para recarregar o Vagrant e observar que a nova pasta foi configurada no Ubuntu além da "vagrant".

Também é interessante não deixar esta última acessível com o arquivo "Vagrantfile" à todos pela máquina virtual. Para desabilitar esta pasta padrão, vemos na parte "Disabling" da documentação o código que devemos inserir:

```
    config.vm.synced_folder ".", "/vagrant", disabled: true
end
```

Salva a alteração do documento no editor de texto, podemos nos conectar com `vagrant ssh`.

Agora, automatizamos a adição da chave pública que geramos anteriormente substituindo o trecho `"echo Hello, World >> hello.txt` por `cat /configs/id_bionic.pub >> .ssh/authorized_keys` para imprimir o conteúdo e jogar a saída para a pasta "Authorized_keys" ao mesmo tempo que garante a presença da chave padrão, obtendo no arquivo "Vagrantfile":

```
    config.vm.provision "shell",
        inline: "cat /configs/id_bionic.pub >> .ssh/authorized_keys"
```

O próximo passo é destruir e refazer a máquina virtual com `vagrant destroy -f && vagrant up` no terminal. Terminada a operação, vemos o diretório com as configurações, o Shell e o `inline script` sendo executados.

Nos conectamos mais uma vez para verificar se a inserção automática das chaves ocorreu escrevendo `cat .ssh/authorized_keys`.

Saia com `exit`, apague o arquivo "known_hosts" da pasta ".ssh" e refaça a conexão com a chave privada e o IP digitando `ssh -i id_bionic vagrant @192.168.1.24` no terminal.

Na próxima etapa falaremos mais sobre o Shell Provisioner e suas variações para preparar o MySQL, tendo um exemplo mais prático de nossas configurações.



Provisionando o MYSQL
Continuando com o Shell Provisioner, iremos preparar nossa máquina virtual para instalar o **MySQL**.

No editor de texto com o "Vagrantfile" aberto, copiamos a sentença que configura o Shell e substituímos na cópia para possibilitar a instalação específica do pacote, obtendo:

```
    config.vm.provision "shell",
        inline: "cat /configs/id_bionic.pub >> .ssh/authorized_keys"
    config.vm.provision "shell",
        inline: "apt-get update && apt-get install -y mysql-server-5.7"
```

Estes comandos são executados automaticamente como `sudo`, ou seja, já temos os direitos administrativos como administradores. Com isso, podemos configurar o MySQL e criar um banco de dados, usuário padrão etc.

Observando a documentação, vemos outras formas de escrever este `script`. De volta à parte "Shell Provisioner" dentro de "Provisioning", temos a variável `$script` definida para ser adicionada e alterada no começo do texto do código de "Vagrantfile", bem como a adição de um usuário que pode acessar a máquina virtual a partir de qualquer host com `%`:

```
$script_mysql = <<-SCRIPT
    apt-get update && \
    apt-get install -y mysql-server-5.7 && \
    mysql -e "create user 'phpuser'@'%' identified by 'pass';"
SCRIPT
```

No trecho antes ocupado por `apt-get update && apt-get install -y mysql-server-5.7` na configuração de `shell`, substitua pela variável `$script_mysql`.

Em seguida, destruiremos e recriaremos a máquina virtual no terminal com `vagrant destroy -f && vagrant up`. Na saída, temos bastante informações extras porque o programa autorizou a instalação do pacote. Podemos verificar conectando com `vagrant ssh` e executando o MySQL com `sudo mysql`.

Na sequência, pedimos ao programa que nos mostre os usuários cadastrados escrevendo `select user from mysql.user`. Com a verificação feita, saia com `exit`.

Por enquanto, este MySQL só aceita conexões a partir desta máquina virtual específica. Logo, devemos automatizar o acesso vendo o conteúdo da pasta `cat /etc/mysql/mysql.conf.d/mysqld.cnf` que possui o IP `bind-address = 127.0.0.1`.

Devemos jogar esta saída para a pasta "configs", escrevendo e executando o comando `cat /etc/mysql/mysql.conf.d/mysqld.cnf >> /configs/mysqld.cnf` criando um novo arquivo no Windows.

Com o documento "mysqld.cnf" aberto no editor de texto, busque pela linha que contém o IP apresentado e altere para `bind-address = 0.0.0.0` para aceitar conexão de outros IP. Salve e retorne ao "Vagrantfile" e adicione inclusive a reinicialização do MySQL:

```
    config.vm.provision "shell",
        inline: "cat /configs/id_bionic.pub >> .ssh/authorized_keys"
    config.vm.provision "shell", inline: $script_mysql
    config.vm.provision "shell",
        inline: "cat /configs/mysqld.cnf > /etc/mysql/mysql.conf.d/mysqld.cnf"
    config.vm.provision "shell",
        inline: "service mysql restart"
```

Testamos novamente destruindo e reconstruindo a máquina virtual com a mesma metodologia que já conhecemos. Desta forma, conseguimos instalar o MySQL com o Shell script.

Se executarmos `vagrant provision` no terminal e este comando apresentar alterações indesejadas na máquina virtual, devemos revisar e aprimorar os scripts e provisionamentos.

Se já atuamos como administradores, podemos chamar o "External Script" presente na documentação para adicionar no Vagrantfile ou em seu próprio repositório de códigos.

Na próxima etapa, veremos como executar outras funcionalidades de provisionamento exercidas pelo Shell com outras ferramentas.