Configurações do Provedor

Configurações do provider
Falaremos sobre configurações específicas do nosso provedor.

Abrimos o Oracle VM VirtualBox Gerenciador para visualizar as máquinas configuradas através do "Vagrantfile" que consomem recursos do sistema. Podemos simplificar nosso trabalho com alguns ajustes.

Em "Configuration" de "Virtual Box" na documentação do Vagrant, vemos ao final o código que define os recursos básicos e otimiza o uso de memória com algumas alterações:

```
Vagrant.configure ("2") do |config|
    config.vm.box = "ubuntu/bionic64"

    config.vm.provider "virtualbox" do |vb|
    vb.memory = 512
    vb.cpus = 1
end
```

Execute todas as máquinas com `vagrant up` e observe-as no Oracle VM VirtualBox com os valores de Memória Principal do Sistema definidos anteriormente. Estas últimas configurações podem ser específicas para cada máquina virtual copiando e colando o trecho nos blocos de cada uma, alterando os dados conforme necessidade. Por exemplo:

```
    config.vm.define "phpweb" do |phpweb|
        phpweb.vm.network "forwarded_port", guest: 8888, host: 8888
        phpweb.vm.network "public_network", ip: "192.168.1.25"

        phpweb.vm.provider "virtualbox" do |vb|
            vb.memory = 1024
            vb.cpus = 2
            vb.name = "ubuntu_bionic_php7"
        end
```

Em seguida, digite `vagrant reload phpweb` no terminal para ver as alterações no Oracle VM VirtualBox.

No próximo vídeo, veremos como definir pacotes para cada máquina virtual.



Vagrant Box e Global Status
Veremos como ter versões de sistemas operacionais diferentes para cada máquina virtual através de _boxes_ distintos.

Abra outra janela de terminal **Git Bash** e localize a pasta raiz do usuário com `pwd`, seguido de `vagrant status` para receber a informação de que este ambiente está vazio, pois só funciona dentro de "Vagrantfile".

Mas também o terminal nos devolve uma dica bastante útil que nos permite ver todas as máquinas que estão rodando no sistema com `vagrant global-status`. Ao executar esta ação, recebemos a saída com todas os ambientes existentes atualmente em nosso computador, comandos que funcionam a partir de qualquer pasta se passada a ID individual e limpeza da tabela.

Limpe a tela e insira `vagrant global-status --prune` para vermos a saída apenas com as três máquinas virtuais configuradas devidamente validadas. Podemos executar `halt` a partir de qualquer pasta adicionando a ID apresentada na tabela, como `vagrant halt bc0947c` da `ansible`.

Para podermos configurar o pacote para cada MV, vá ao final do documento "Vagrantfile" e escreva como se fôssemos usar uma versão mais leve de Linux. Para isso, vá ao Vagrant Cloud em seu navegador e busque por "CentOS 7" para acessar as configurações apresentadas e inserir:

```
    config.vm.define "memcached" do |memcached|
        memcached.vm.box = "centos/7"
        memcached.vm.provider "virtualbox" do |vb|
            vb.memory = 512
            vb.cpus = 1
            vb.name = "centos7_memcached"
        end
```

De volta ao terminal com a pasta `/d/ambiente_dev/bionic` aberta, o `vagrant status` deve mostrar a nova máquina virtual configurada. Para mostrar a diferença, o `vagrant global-status` não a apresenta pois ainda não a subimos.

Então, comande `vagrant up memcached` para adicioná-la. Inclusive, esta também é exibida no Oracle VM VirtualBox.

Para vermos quais os boxes que temos instalados, basta escrever `vagrant` para ver todos os comandos do programa e buscar por `box`. Desta forma, escreva `vagrant box list` para ver os pacotes, retire `hashicorp/precise64` digitando `vagrant box remove harshicorp/precise64` e remova os desatualizados e/ou duplicados com `vagrant box prune`.

Assim, sobram somente `cento/7` e `ubuntu/bionic64`.

No próximo passo, veremos a comparação entre o Vagrant junto com o VirtualBox e o **Docker**.


Nesta aula, aprendemos que:

-   No **Vagrantfile**, podemos definir configurações específicas do provedor (`hypervisor`)
-   As configurações são referente à memória, CPU, rede ou interface gráfica, entre outras opções
-   Para listar todos as _boxes_ baixadas, use o comando: `vagrant box list`
-   Para remover as boxes desatualizadas: `vagrant box prune` ou `vagrant box remove <nome>`
-   Para listar todas as máquinas que foram criadas no _host_, use: `vagrant global-status --prune`
-   Através do ID da máquina, podemos controlar a máquina virtual fora da pasta do projeto, por exemplo: `vagrant destroy -f <ID-da-VM>`