Lidando com o SSH

Adicionando a chave SSH
Com as configurações de rede definidas, falaremos novamente sobre a conexão SSH neste passo.

Vimos que a motivação para instalar uma máquina virtual que comporte bancos de dados e outras ferramentas através do Vagrant é a de manter seu sistema operacional limpo, possibilitando a fácil exclusão e recriação conforme a necessidade.

Também aprendemos como disponibilizá-la em uma rede empresarial por meio da Public Network para uso coletivo dos desenvolvedores. Desta forma, é importante estabelecer este acesso remotamente.

Se escrevermos somente `vagrant ssh-config`, vemos que só é possível acessar via SSH a máquina virtual através do localHost por enquanto. Para que outros equipamentos também consigam precisamos ter um IP fixo para testarmos escrevendo `ssh vagrant@192.168.1.24` no terminal.

Uma mensagem de erro é apresentada porque o SSH automaticamente guarda a identificação das máquinas usadas para nos conectarmos em um arquivo chamado "known_hosts".

Ainda nesta linha de comando, copie e cole o endereço deste diretório que foi apresentado após escrever `cat`; repare que o arquivo guarda o IP e uma identificação específica. Geralmente, o sistema nos pergunta se queremos adicioná-la ao documento e respondemos positivamente quando nos conectamos pela primeira vez.

Porém, em nosso caso, a ID mudou por termos recriado a máquina através do Vagrant, sendo necessário que apaguemos o arquivo apresentado. Para isso, digite `rm` seguido do endereço do mesmo para executar.

Limpe a tela e teste mais uma vez escrevendo `ssh vagrant@192.168.1.24` para responder com `yes` à adição da nova identificação. O terminal nos alerta que a permissão da chave pública foi negada para este comando SSH que nos conecta remotamente, então não podemos usar esta fórmula.

Logo, é muito comum usarmos uma chave pública e outra privada para nos conectarmos. Quando criamos a máquina virtual, o Vagrant automaticamente gera uma chave pública e a guarda na pasta "authorized_keys" dentro de ".ssh" quando inserimos o comando `cat .ssh/authorized_keys` no terminal.

Olhando novamente a partir do Windows, ao escrever `vagrant ssh-config` aparece que não é necessária a utilização de senha para autenticação no LocalHost, pois o programa usa os dois tipos de chave. Nada impede que utilizemos a chave privada para conexão digitando `ssh -i .vagrant/machines/default/virtualbox/private_key vagrant@192.168.1.24` usando autenticação em ambas como vimos na aula sobre PuTTY.

Para outros desenvolvedores se conectarem remotamente à máquina virtual, devem receber a chave privada.

No próximo passo, veremos como gerar nossa própria chave privada e como a adicionamos à pasta de chaves públicas para ser usada.


Gerando uma chave SSH
Neste passo, veremos como gerar nossa própria chave privada para permitir o acesso de outros desenvolvedores à máquina virtual.

Começamos encerrando com `vagrant@ubuntu-bionic:~$ exit` no terminal e retornando ao Windows escrevendo `vagrant status`. Para criar a chave, utilizamos o PuTTY ou, caso já tenha o SSH instalado no sistema operacional, existe uma metodologia bem fácil.

Insira o comando `ssh-keygen -t rsa` para escolher a pasta onde será salva. Aqui, basta escrever `/d/ambiente_dev/bionic/id_bionic`. Depois o sistema nos pede para criar uma senha de acesso para protegê-la caso queira, gerando uma imagem com caracteres para gerar a chave. Em seguida, ao limparmos a tela e escrevermos `ls`, ambas nos são apresentadas.

Agora precisamos copiar nossa chave pública para a máquina virtual; começamos por digitar `vagrant ssh` e acessar a pasta com `ls /vagrant/` que é automaticamente montada no Windows.

Copiamos a chave publica da pasta "vagrant" para a máquina virtual escrevendo `cp /vagrant/id_bionic.pub .`, sendo possível ser visualizada com `ls` novamente. Precisamos adicioná-la ao arquivo "authorized_keys" digitando `cat id_bionic.pub >> .ssh/authorized_keys`. Veja a nova chave adicionada com `cat .ssh/authorized_keys`. Finalize com `exit` para retornar ao Windows.

Agora, usaremos a chave privada no Windows para nos conectar usando `ssh -i id_bionic vagrant@192.168.1.24`, lembrando que o seu IP pode variar. É possível assim se conectar remotamente com sua máquina virtual baseado em uma chave pública/privada desde que as tenha associadas com a chave privada dentro da MV.

No próximo passo veremos sobre automação e _provisioners_.


Nesta aula, aprendemos que:

-   O comando `vagrant ssh-config` lista as configurações SSH que o comando `vagrant ssh` usará
-   O **Vagrant** gera automaticamente um par de chaves SSH
-   A chave pública fica na máquina virtual (_guest_), a chave privada fica no _host_
-   No arquivo **.ssh/known_host**, fica guardado o _fingerprint_ de cada máquina com qual o SSH se conectou
-   Como criamos máquinas através do Vagrant com frequência, é preciso limpar esse arquivo **.ssh/known_host** (ou apagar) de tempos e tempos
-   Para gerar um par de chaves SSH, existe a ferramenta `ssh-keygen`
-   A chave pública deve ficar dentro do arquivo **.ssh/authorized_keys** da máquina virtual