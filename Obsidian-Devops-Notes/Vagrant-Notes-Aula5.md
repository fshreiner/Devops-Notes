Conhecendo o Puppet

Ambiente Multi-machine
Nos organizaremos melhor criando uma nova máquina virtual a partir do mesmo documento "Vagrantfile". Com o MySQL já configurado, instalaremos também o **PHP**.

No tópico "Multi-Machine" presente na documentação do Vagrant, temos as informações para gerar múltiplos ambientes. Copie a última linha de código apresentada, altere e adicione ao arquivo:

```
Vagrant.configure ("2") do |config|
    config.vm.box = "ubuntu/bionic64"

    config.vm.define "mysqldb" do |mysql|
```

Todo o bloco subsequente é recortado e colado logo após este último bloco do MySQL que inserimos, alterando os prefixos `config` de cada configuração para `mysql`, garantindo o encapsulamento do programa. Ou seja, quando subirmos a máquina, está não terá mais a nomenclatura `default`, passando a se chamar `mysqldb`.

Apagamos a sentença que estabelece a rede `forwoarded_port` e deixamos somente a chave pública com a IP.

Voltando à documentação, copiamos a linha de código intermediária, alteramos alguns elementos e implementamos ao final do "Vagrantfile". Definimos uma IP diferente para esta nova máquina virtual que será baseada no Ubuntu:

```
    config.vm.define "phpweb" do |phpweb|
        phpweb.vm.network "forwarded_port", guest:80, host:8089
        phpweb.vm.network "public_network", ip: "192.168.1.25"
    end
end
```

Com a MV destruída na aula anterior, testamos a presença de duas novas máquinas ao executar `vagrant status` no terminal: `mysqldb` e `phpweb`. Agora, caso queira subir somente uma delas, basta escrever `vagrant up mysqldb` por exemplo. Todos os outros comandos podem ser executados neste mesmo modelo, adicionando o nome ao final. Quado quisermos acionar todas, basta apenas não incluir o nome específico.

Agora, digitamos somente `vagrant up` e aguardamos o fim do processo. Teste a conexão SSH com ambas e depois observe suas configurações com `vagrant ssh-config`; repare que são utilizadas duas portas diferentes.



Usando o Puppet
Neste passo, falaremos sobre outros provisioners além do Shell; na documentação, temos vários itens na parte de "Provisioning" que apresentam ferramentas para facilitar a configuração da máquina virtual.

Começamos pela instalação do **Puppet Apply Provisioner** na `phpweb` para configurá-la, e você pode encontrar mais informações e sua declaração [nesta página](http://www.digitalocean.com/community/tutorials/getting-started-with-puppet-code-manifests-and-modules). É preciso definir os _manifests files_ em um novo diretório chamado "manifests" dentro de "configs", e criar um novo arquivo chamado "phpweb.pp" no editor de texto onde adicionamos e salvamos, além de criar uma nova pasta chamada "src" que receberá um novo documento "index.pp" que ficará em branco por enquanto:

```
# execute 'apt-get update'
exec { 'apt-update':
    command => '/usr/bin/apt-get update' 
}

package { ['php7.2' ,'php7.2-mysql'] :
    require => Exec['apt-update'],
    ensure => installed,
}

exec { 'run-php7':
    require => Package['php7.2'],
    command => '/usr/bin/php -S 192.168.1.25:8888 -t /vagrant/src &'
}
```

Para testar, vá ao terminal e insira `vagrant ssh phpweb` para chamar este novo arquivo com a entrada `ls /vagrant/configs/manifests/`.

Precisamos instalar o Puppet executando `sudo apt-get update && sudo apt-get install -y puppet`.

Limpe a tela e ative-o com `sudo puppet apply /vagrant/configs/manifests/phpweb.pp`.

Para testar a execução do PHP, devemos realizar alguns ajustes que veremos no passo seguinte.



Integração do Puppet com o Vagrant
Faremos a integração do Puppet com o Vagrant nesta etapa.

Com "Vagrantfile" aberto no editor de texto, usamos uma técnica de instalação a partir do Shell e inserimos o "phpweb.pp". Além disso, recuperamos a linha apresentada na documentação em "Puppet Apply Provisioner" fazendo algumas personalizações, adicionando ao final do código:

```
        phpweb.vm.provision "shell",
            inline: "apt-get update && apt-get install -y puppet"

        phpweb.vm.provision "puppet" do |puppet|
            puppet.manifests_path = "./configs/manifests"
            puppet.manifest_file = "phpweb.pp"
        end
    end
end
```

Altere também os elementos para `guest:8888, host: 8888`, repetindo o procedimento em "phpweb.pp" com IP acessível pelo MySQL:

```
exec { 'run-php7':
    require => Package['php7.2'],
    command => '/usr/bin/php -S 0.0.0.0:8888 -t /vagrant/src &'
```

No terminal simulador de Linux, destruiremos e recriaremos a máquina virtual do `phpweb` escrevendo `vagrant destroy -f phpweb && vagrant up phpweb`. Verificamos se o PHP está rodando com `vagrant ssh phpweb`, seguido de `netstat -tln` e `ps aux | grep php` para encontrar o processo.

Abra o documento "index.pp" criado anteriormente dentro da pasta "src" e digite para depois testar no navegador inserindo `localhost:8888` e `192.168.1.25:8888`:

```
<?php
echo "Funcionou!!!"
?>
```

Para testar a conexão, apague o código anterior de "index.php" e insira:

```
<?php
echo "Testando conexao <br /> <br />";
$servername = "192.168.1.24";
$username = "phpuser";
$password = "pass";

// Create connection
$conn = new mysqli($servername, $username, $password);

// Check connection
if ($conn->connect_error) {
    die("Conexão falhou: " . $conn->connect_error);
}
echo "Conectado com sucesso";
?>
```

Assim, temos uma integração concluída. Caso haja interesse, recomendamos que aprofunde os conhecimentos sobre Puppet com os cursos oferecidos pela plataforma **Alura**.

No próximo passo, veremos sobre a integração com o **Ansible**.


Nesta aula, aprendemos que:

-   No mesmo **Vagrantfile**, podemos configurar várias máquinas, separando as configurações (_Multi-Machine_)
-   O **Puppet** é uma ferramenta popular para provisionar uma máquina
-   Provisionamento significa instalar e configurar tudo o que for necessário para rodar algum serviço ou aplicação
-   Com **Puppet**, podemos definir os passos de instalação de mais alto nível, facilitando a manutenção
-   Os passos de instalação são configurados em um arquivo **manifest**, com a extensão **.pp**
-   Para rodar o **Puppet**, é preciso instalar um cliente na máquina virtual
-   O **Vagrant** integra e consegue chamar o **Puppet** a partir do comando `vagrant provision`
-   Ao rodar o comando `vagrant up` pela primeira vez, ele também roda o provisionamento
-   Para configurar o Puppet dentro do **Vagrantfile**, basta usar:
    
    ```
    config.vm.provision "puppet" do |puppet|
      puppet.manifests_path = "manifests"
      puppet.manifest_file = "wep.pp"
    end
    ```