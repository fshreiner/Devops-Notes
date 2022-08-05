![[Pasted image 20220802205719.png]]

Isolização de ambientes!

Durante o dia a dia de desenvolvimento é muito melhor separar cada ambiente solicitado por cada aplicação, é muito mais vantajoso do que ficar instalando várias versões do mesmo programa.

A melhor forma de fazer isso, é usando da virtualização pois assim não é necessário ter vários hardwares físicos, um para cada aplicação.

O Vagrant faz o controle do virtualizador que você utiliza, por exemplo você está utilizando o virtualbox, e pode usar o Vagrant para criar os arquivos de configuração das máquinas, assim quando tiver que replicar, está tudo pronto.

O Vagrant faz literalmente a Infraestrutura por código.



Fazendo a Instalação do Vagrant

Você pode encontrar o software neste [endereço](http://www.vagrantup.com/). Logo no início, há o botão de _download_ que leva à página onde podemos escolher a versão e o sistema operacional Windows. Sugerimos que utilize a mais próxima deste curso.

Como o Vagrant não pode funcionar sozinho, precisamos de um provedor. Na [documentação](http://www.vagrantup.com/docs/providers), podemos ver quais são as versões compatíveis do VirtualBox que usaremos, o qual pode ser baixado [aqui](http://www.virtualbox.org/) para selecionarmos "Windows hosts" e o pacote de expansão para todas as plataformas. Abra o instalador deste último para prosseguir até a conclusão, e depois instale o Vagrant.

Como este programa atua na linha de comando, abra o Prompt de Comando ou Terminal. Insira `vagrant version` na continuação da última linha escrita para checarmos se a instalação ocorreu corretamente.

Para verificar a integração com o VirtualBox, mude para o drive `D:` e crie uma pasta de nome "ambiente_dev" executando `mkdir`. Dentro desta, digite `vagrant init hashicorp/precise64` e tecle "Enter" para visualizar a mensagem que indica a criação do arquivo neste diretório e a confirmação da instalação deste _box_.

Ao pesquisar no buscador o trecho "precise64", o primeiro link direciona para esta [página](https://app.vagrantup.com/hashicorp/boxes/precise64) onde encontramos algumas configurações.

Vá até a pasta "ambiente_dev" e abra o documento "Vagrantfile" com o editor de texto, no caso usamos o **Atom**. Neste, encontramos a sentença de configuração `Vagrant.configure("2")` e `config.vm.box = "hashicorp/precise64`.

Ainda, o programa nos diz que cada ambiente de desenvolvimento necessita de um _box_, ou seja, um pacote que contém as informações necessárias para rodar uma máquina virtual e nos apresenta um [link de busca](http://vagrantcloud.com/search) onde podemos encontrar a versão mais recente. Porém, já o fizemos ao digitar a segunda configuração apresentada pelo programa.

No terminal, digite `vagrant up` para subir a máquina virtual, o que pode demorar algum tempo no primeiro momento já que o software está baixando o pacote da internet. Finalizado este processo, é criada uma nova pasta ".vagrant" dentro de "ambiente_dev", onde encontramos mais informações.

De volta ao Prompt, escreva `vagrant status` para verificar se está sendo executado corretamente por meio do retorno que o sistema nos dá.



Acessando a Máquina Virtual

Com o Vagrant e o VirtualBox instalados e a máquina virtual rodando no terminal, abra o aplicativo **Oracle VM VirtualBox** para visualizar o programa em modo de execução na barra lateral.

Agora, precisamos nos conectar através do **SSH**. Em Linux, este comando costuma ser executado sem problemas ao digitarmos no prompt `vagrant ssh`; mas em Windows podemos ter algumas questões.

Caso a ação não ocorra facilmente, vá nas configurações do sistema e clique na opção "Aplicativos" para acessar "Aplicativos e recursos". Clique sobre "Gerenciar recursos opcionais" e verifique se na lista apresentada há o item "Cliente OpenSSH". Se não estiver, existem duas opções:

A primeira que usaremos neste curso, é a instalação de um terminal que simula o do Linux a partir do controle de versão **Git BASH** encontrado neste [link](http://gitforwindows.org/). Faça o download, conclua o processo e abra-o, pois passaremos a utilizar majoritariamente este.

Neste, acesse o diretório do projeto escrevendo `cd /d/ambiente_dev/`, tecle "Enter" e escreva `pwd` para aceder o arquivo. Por fim, podemos executar o comando anterior digitando `vagrant ssh` para conectar.


Para saber mais: Tipos de Hypervisor

Um _Hypervisor_, também conhecido como monitor de máquina virtual, é um processo que cria e executa máquinas virtuais (VMs). Um _Hypervisor_ permite que um computador _host_ suporte múltiplas VMs, compartilhando virtualmente seus recursos, como memória e processamento.

Exemplos de _Hypervisors_ são:

-   Hyper-V
-   vSphere
-   Parallels
-   VMware
-   Virtualbox
-   entre outros.

Existem dois _tipos de Hypervisors_: **Tipo 1** e **Tipo 2**.

Os _Hypervisors_ do **Tipo 1** são chamados de "bare metal", pois são executados diretamente no hardware do _host_. Exemplos disso são **Hyper-V** e **vSphere** (entre vários outros).

Os _Hypervisors_ do **Tipo 2** rodam como uma aplicação em cima do sistema operacional. Exemplos são o **VirtualBox** e **VMware**.

Nesta aula, aprendemos que:

-   **VirtualBox**, **VMware**, **Hyper-V**, entre outros, são **_Hypervisors_**
-   Um _Hypervisor_ emula o hardware do computador para criar e executar máquinas virtuais
-   O **Vagrant** é uma ferramenta que controla o _Hypervisor_ a partir de um arquivo simples, o **Vagrantfile**
-   O **Vagrantfile** define detalhes da máquina virtual, como o sistema operacional, a rede, software utilizado, etc
-   O comando `vagrant init <box>` cria um **Vagrantfile**
-   A box é baixada da internet e possui a imagem do sistema operacional, entre outras configurações
-   Para inicializar e rodar a VM com Vagrant, usa-se o comando: `vagrant up`
-   O comando `vagrant status` mostra detalhes sobre o status da máquina virtual
-   Para se conectar com a máquina virtual, usamos a ferramenta SSH


