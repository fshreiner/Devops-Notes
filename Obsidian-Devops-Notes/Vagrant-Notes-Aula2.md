Inicio Port Fowarding

Quando digitamos no terminal que simula Linux a sentença `cat Vagrantfile`, podemos ver um grande texto com muitos comentários. Entenderemos melhor sobre este arquivo e configuração de rede neste passo.

Para pararmos uma máquina virtual, escrevemos `vagrant halt`. Para confirmar, digitamos `vagrant status`.

Também abordaremos a configuração da rede para acessar algum serviço rodando dentro da máquina virtual e testar uma aplicação, como um banco de dados por exemplo. Se este apresenta problemas para ser executado, devemos criar um ambiente de desenvolvimento eficaz.

Antes disso, vamos nos organizar; no diretório "ambiente_dev", crie uma nova pasta nomeando como "bionic" e outra de nome "precise", deslocando "Vagrantfile" e ".vagrant" para dentro desta última.

A seguir, entre [neste site](http://app.vagrantup.com/ubuntu) para acessar a última versão de "ubuntu/bionic64". Clicando neste, copie o bloco de código na caixa "vagrantfile" e cole em um novo arquivo no editor de texto que será salvo no diretório "bionic".

```
Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/bionic64"
end
```

Com o Vagrantfile configurado, retorne ao terminal e limpe-o se necessário digitando `clear`. Escreva `ls` para visualizar as duas pastas em "ambiente_dev" e tecle "Enter", depois escreva `cd bionic` e tecle novamente, e por fim digite `ls` novamente para abrir o arquivo "Vagrantfile". Em seguida, escreva o comando `vagrant up` para executar a importação do pacote, e é possível que demore algum tempo até a conclusão.

Enquanto isso, é interessante deixar aberta a página da documentação do Vagrant que contém informações sobre _Networking_ clicando [aqui](http://www.vagrantup.com/docs/networking).

Terminada a importação, vá ao terminal, limpe a tela com `clear` e verifique a atividade do Vagrant escrevendo `vagrant status`. Se estiver rodando na máquina virtual, já podemos nos conectar digitando `vagrant ssh`; repare que há algumas diferenças, pois estamos executando a última versão de **Ubuntu**. limpe novamente.

Agora, se quisermos instalar um serviço como o servidor Nginx por exemplo, devemos executar duas coisas como administrador: a primeira é a atualização da lista de pacotes escrevendo `sudo apt-get update` e a segunda é a instalação do próprio programa digitando `sudo apt-get install -y nginx`. Terminadas as duas operações temos o Nginx teoricamente rodando. Limpe a tela novamente.

Para confirmar e ver o número do processo, digite `netstat -lntp` e tecle "Enter". Há outro comando de envio de requisição chamado `curl http://localhost` que nos devolve em html.

Para acessar corretamente a máquina virtual Linux com a porta que desejamos, devemos realizar algumas configurações. Para isso, voltamos ao endereço que contém a documentação do Vagrant, lemos a parte de "Forwarded Ports" e vemos que podemos usar uma porta específica em Windows, o qual recebe a requisição e acessa a máquina virtual automaticamente através do processo de **_Port Forwarding_** na rede.

Para definir esta chamada, adicione a linha disponibilizada na página ao arquivo "Vagrantfile" no editor de texto, alterando para o host do Windows e salvando:

```
Vagrant.configure("2") do |config|
    config.vm.network "forwarded_port", guest: 80, host:8080
end
```

No terminal, pare a máquina virtual escrevendo `vagrant halt` para recarregar esta configuração com `vagrant up`. Vá ao navegador e digite `localhost:8089` para delegar a função à máquina virtual. Ao dar "Enter", nos é exibido a página html do Nginx que vimos na linha de comando. Repare que, ao clicar com o botão direito e selecionar "Exibir código fonte da página", vemos o mesmo código de nossa **vm**.

Então, quando quiser realizar o _Port Forwarding_ e acessar o banco de dados de um MySQL por exemplo, basta inserir a porta do programa e a porta que quer utilizar para rodar a aplicação no Windows.

No próximo passo, veremos como configurar uma IP estática para acessarmos nossa máquina virtual através de um aplicativo específico.


Já configuramos o acesso a partir do processo de Port Forwarding. Em muitos casos, é preferível que a máquina virtual possua um IP a ser usado no código para referenciar o MySQL, um servidor de web e etc., permitindo que seja recriada em outros ambientes ao reaproveitar sua identificação.

Confirmada a execução da MV no terminal, voltamos à documentação do Vagrant para acessarmos a parte "Static IP" em "Network > Private Network". Copiamos o trecho de código que configura a rede e colamos no arquivo "Vagrantfile" no editor de texto.

```
config.vm.network "private_network", ip: "192.168.50.4"
end
```

No terminal, paramos a máquina virtual escrevendo `vagrant halt` para executar novamente com `vagrant reload`. Nem sempre este último comando funciona perfeitamente, demandando alterações no sistema operacional Windows. Caso este erro seja apresentado, falaremos mais adiante na próxima aula sobre como resolver essa situação.

Se tudo ocorrer como esperado, observamos o produto do recarregamento e notamos que o `Adapter 1` configurado que acessa a rede é a base para SSH e Port Forwarding. Já o `Adapter 2` é resultado da configuração que acabamos de implementar.

Isso indica mudanças no sistema operacional Windows que podem ser vistos inserindo `ipconfig` para visualizar as duas interfaces de rede. Não podemos confundir estas com os adaptadores da máquina virtual mostrados anteriormente.

O primeiro "Ethernet VirtualBox Host-Only Network" foi criado pelo VirtualBox para definir a rede da MV, e possui uma mascara relacionada com os números do IP da configuração retirada da documentação. Ou seja, se limparmos a tela e acessarmos digitando `vagrant ssh` que utiliza apenas o `Adapter 1`, observamos a presença da linha "IP address for enp0s8: 192.168.50.4" que indica a existência de um IP estático.

Isso pode ser provado também executando o comando `ifconfig` que nos apresenta três adaptadores, sendo um destes o que estabelece o IP estático.

Abra o Prompt de Comando do Windows para não perder a sessão configurada no outro terminal e digite `ping 192.168.50.4` para a máquina virtual com este número de IP.

Ou seja, no navegador podemos usar este IP para acessar a MV e o servidor web Nginx diretamente em outra aba ao mesmo tempo que usamos o Port Forwarding com `localhost:8089` na aba anterior.

Para demonstrar melhor o processo, acesse a interface "Oracle VM VirtualBox" e repare que há as duas máquinas virtuais na lista. Selecione "bionic_default" e clique em "Preferências > Configurações" para abrir uma nova caixa de diálogo. Nesta, clique em "Rede" para ver os dois adaptadores; nos campos, as informações batem com as do Prompt Windows.

O VirtualBox cria uma mudança no sistema operacional Windows e na máquina virtual para ter a mesma identificação nas redes, fazendo ambas funcionarem.

Indo em "Arquivo(F) > Host Network Manager...", abrimos uma janela chamada "Gerenciador de Redes do Hospedeiro" onde podemos observar o adaptador "Ethernet VirtualBox Host-Only Network" configurado com as alterações no sistema que permitem o acesso à MV.

No próximo vídeo, veremos as configurações de "DHCP" presentes na documentação e como resolver a questão de recarregar o Vagrant destruindo a máquina virtual e refazendo-a.
