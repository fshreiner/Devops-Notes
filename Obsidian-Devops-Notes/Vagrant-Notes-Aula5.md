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



Integração do Puppet com o Vagrant