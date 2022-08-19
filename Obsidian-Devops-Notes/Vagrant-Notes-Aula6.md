Usando o Ansible


Introdução ao Ansible
Integraremos mais o provisionador **Ansible**.

Nos passos anteriores, começamos a abordagem sobre este recurso com o Shell Provisioner que envia o código para a máquina virtual permitindo a execução de comandos e a instalação do MySQL nesta com base no pacote Ubuntu Bionic 64, usando `vagrant up mysqldb`. Porém, vimos que há ferramentas mais fáceis de se trabalhar.

Com isso, utilizamos o Puppet para receber a última versão do PHP também com a mesma base, gerando um documento "phpweb.pp" com os passos de instalação que gostaríamos, subindo com `vagrant up phpweb`.

Para usarmos Ansible, também o executamos na máquina `host`, ou seja, a partir do nosso sistema operacional Windows, da mesma forma que o Vagrant. Definimos os processos em um documento "playbook.yml" análogo aos arquivos `.pp`, o qual envia os comandos para a máquina virtual e não exige sua instalação nesta `guest`.

A distinção desta última ferramenta apenas precisa do **Python** instalado, o que é comum em sistemas Linux. No entanto, o Ansible não roda no Windows, somente em **Mac** ou Linux, diferente do Vagrant que é habilitado para todos.

Logo, precisamos criar uma máquina virtual para executar o Ansible e reconfigurar o servidor MySQL. Caso possua Linux, não é preciso criar outra MV.

Veremos os procedimentos completos na próxima atividade.


Instalação do Ansible
Prepararemos nossa máquina virtual para receber o Ansible.

Garanta que nenhuma esteja rodando ao verificar com `vagrant status` no terminal.

Vá ao final do documento "Vagrantfile" no editor de texto para criarmos uma nova MV, reconfigurarmos o **MySQL Server** e instalarmos o Ansible e suas redes com IP diferentes.

Devemos utilizar o Shell para instalar o Ansible, visto que este não é recebido automaticamente. Clique [neste endereço](http://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) para acessar o link "Latest Releases via Apt (Ubuntu)" e copiar as quatro linhas de código apresentadas que devem ser introduzidas na sequência.

```
    config.vm.define "mysqlserver" do |mysqlserver|
        mysqlserver.vm.network "public_network", ip: "192.168.1.22"
    end

    config.vm.define "ansible" do |ansible|
        ansible.vm.network "public_network", ip: "192.168.1.26"
        ansible.vm.provision "shell",
            inline: "apt-get update && \
                     apt-get install -y software-properties-common && \
                     apt-add-repository --yes --update ppa:ansible/ansible && \
                     apt-get install -y ansible"
    end
end
```

Testamos as alterações com `vagrant validate` seguido de `vagrant status` para visualizar as quatro máquinas virtuais e ativar com `vagrant up ansible`.

Acessamos a máquina em execução com `vagrant ssh ansible` e a visualizamos com `ansible-playbook --version`.

Agora no "Vagrantfile", passamos os passos de instalação pelo arquivo "playbook.yml" que envia comandos à MV a ser configurada com chave privada para instalar o MySQL Server com chave pública através da conexão SSH entre estas. Para isso, obtemos:

```
    config.vm.define "mysqlserver" do |mysqlserver|
        mysqlserver.vm.network "public_network", ip: "192.168.1.22"

        mysqlserver.vm.provision "shell",
            inline: "cat /vagrant/configs/id_bionic.pub >> .ssh/authorized_keys"
    end
```

Saia com `exit`, execute a máquina do MySQL Server com `vagrant up mysqlserver` e observe os resultados se conectando com `vagrant ssh mysqlserver`.

Como já temos a chave pública em MySQL Server, precisamos adicionar e copiar a chave privada `id_bionic` para habilitar a chamada do Ansible. Também mudamos a permissão para apenas o usuário do arquivo no "Vagrantfile", adicione:

```
config.vm.define "ansible" do |ansible|
    ansible.vm.network "public_network", ip: "192.168.1.26"

    ansible.vm.provision "shell",
        inline: "cp /vagrant/id_bionic  /home/vagrant && \
                chmod 600 /home/vagrant/id_bionic"
```

Testamos com `vagrant provision ansible` e conectamos com `vagrant ssh ansible`. Assim, preparamos a máquina virtual, instalamos o Ansible, colocamos a chave privada à sua disposição e preparamos a MV para o MySQL Server.

Falta-nos apenas executar o "playbook.yml" para fazer todas as configurações necessárias.



Testando o Playbook
Neste passo, nosso objetivo é testar o Ansible.

Dentro de "bionic/configs", crie uma nova pasta no editor de texto com o nome "ansible". Nesta, crie um novo arquivo chamado "hosts" que define qual máquina será configurada com uma lista de IP's e algumas configurações de usuário, conexão, chave privada, permissões e padrão Python.

```
[mysqlserver]
192.168.1.22

[mysqlserver:vars]
ansible_user=vagrant
ansible_ssh_private_key_file=/home/vagrant/id_bionic
ansible_python_interpreter=/usr/bin/python3
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

Salvo este inventário, sabemos qual máquina acessar e como fazê-lo. Agora podemos definir os passos de instalação no novo arquivo "playbook.yml" dentro do mesmo diretório anterior.

```
- hosts: all
  handlers:
    - name: restart mysql
      service:
        name: mysql
        state: restarted
      become: yes

  tasks:
    - name: 'Instalar MySQL Server'
      apt:
        update_cache: yes
        cache_valid_time: 3600 #1 hora
        name: ["mysql-server-5.7", "python3-mysqldb"]
        state: latest
      become: yes

    - name: 'Criar usuario no MySQL'
      mysql_user:
        login_user: root
        name: phpuser
        password: pass
        priv: '*.*:ALL'
        host: '%'
        state: present
      become: yes

    - name: 'Copiar arquivo mysqld.cnf'
      copy:
        src: /vagrant/configs/mysqld.cnf
        dest: /etc/mysql/mysql.conf.d/mysqld.cnf
        owner: root
        group: root
        mode: 0644
      become: yes
      notify:
        - restart mysql
```

Caso queira aprofundar seus conhecimentos em Ansible, recomendamos os cursos da Plataforma Alura acessíveis [aqui](https://cursos.alura.com.br/course/infraestrutura-como-codigo-com-ansible). No terminal, testamos a instalação escrevendo `vagrant ssh ansible` seguido de `ansible-playbook -i /vagrant/configs/ansible/hosts /vagrant/configs/ansible/playbook.yml'`.

Um erro é apresentado indicando que não foi possível se conectar com o IP terminado em `22`. Observamos que ao digitar `ls -al` a saída aponta que a chave `id_bionic` é acessível ao usuário `root`, sendo necessárias alterações no _script_.

Realizamos esta mudança com `sudo chown vagrant:vagrant id_bionic` e verificamos novamente com `ls -al` para atestar a alteração. Copie este penúltimo comando para o "Vagrantfile":

```
        ansible.vm.provision "shell",
            inline: "cp /vagrant/indbionic /home/vagrant && \
                     chmod 600 /home/vagrant/id_bionic && \
                     chown vagrant:vagrant /home/vagrant/id_bionic"
```

Instale o Ansible inserindo `ansible-playbook -i /vagrant/configs/ansible/hosts /vagrant/configs/ansible/playbook.yml` no terminal. Repare que as saídas apresentam as tarefas que estabelecemos e as alterações de IP no "playbook.yml".

Portanto, a partir de nossa máquina virtual podemos controlar o MySQL Server. No próximo passo, faremos a integração do comando no "Vagrantfile" para automatizar a execução.



Integração com o Vagrant
De volta ao terminal com o Windows, verificamos com `vagrant status` que as máquinas virtuais `mysqlserver` e `ansible` estão rodando.

Subimos também o `phpweb` escrevendo `vagrant up phpweb` para testarmos a conexão do servidor MySQL. Conferimos também digitando `localhost:8888` ou `192.168.1.25:8888` no navegador.

Agora, chamaremos o Ansible a partir do arquivo "Vagrantfile" para automatizar sua execução. Para ter acesso às configurações de instalação, vá em "Ansible Provisioner" dentro de "Provisioning" na documentação do Vagrant.

No caso do sistema operacional Windows, é preciso adicionar mais um comando "shell" para executar na máquina virtual. No "Vagrantfile" adicione ao final:

```
        ansible.vm.provision "shell",
            inline: "ansible-playbook -i /vagrant/configs/ansible/hosts \
                 /vagrant/configs/ansible/playbook.yml"
```

Como não estamos mais utilizando a configuração do `mysqldb`, podemos comentar todo seu bloco selecionando-o e indo em "Edit > Toggle Comments". Desta forma, temos apenas três máquinas virtuais definidas.

De volta à linha de comando, insira `vagrant destroy -f && vagrant up` e aguarde a finalização de toda a instalação. Verifique o funcionamento com `vagrant status`.


Mas afinal, qual é a diferença entre _Puppet_ e _Ansible_?

Podemos resumir de alguns modos:

-   O Ansible é uma ferramenta principalmente de **provisionamento**, ou seja, é utilizado para fornecermos ferramentas e preparar nosso ambiente para determinada tarefa.
    
-   Outro fato sobre o Ansible é que tudo que escrevemos em nossos `playbooks` é convertido em código _python_. O que significa que devemos ter o python instalado nas máquinas em que o playbook será executado.
    
-   Os playbooks devem ser executados em cada máquina desejada para execução do serviço, ou seja, para cada vez que desejarmos fazer um novo **provisionamento** para as máquinas, precisamos executar o playbook novamente.
    
-   O Puppet, é uma ferramenta de **gerenciamento de configuração**, ou seja, utilizamos o Puppet para definir e manter as configurações de nosso ambiente.
    
-   Com o Puppet, utilizamos arquivos de `manifest` para definir como será feita e estabelecida a configuração das máquinas que rodarão o _puppet-agent_. Para que isso funcione, devemos ter o puppet-agent instalado em todas as máquinas que serão gerenciadas pelo Puppet, e o _puppet-server_ na máquina que será a provedora de configurações.
    
-   Uma vez definido como as máquinas serão configuradas, executamos o comando para que as máquinas com o puppet-agent comecem a seguir as configurações especificadas em nosso arquivo manifest.
    

Concluindo: o Puppet é uma ferramenta de **gerenciamento de configuração** e o Ansible é uma ferramenta de **provisionamento**, ou seja, utilizamos o Puppet para validar a configuração de nosso ambiente e o Ansible para instalar e preparar o ambiente. Mas como assim, isso não seria **provisionamento** para os dois casos? Na verdade, **`não`**.

Vamos ver um exemplo:

Temos uma máquina e devemos construir o ambiente para nosso trabalho. Como queremos definir as configurações iniciais de uma máquina, seria interessante **provisioná-la** inicialmente, já que sequer temos o que manter de configuração. Depois de definido o ambiente, precisamos manter essas configurações. Caso algum programa ou arquivo seja removido, queremos que o estado da máquina seja restaurado para o estado original, sem afetar o funcionamento . Para garantirmos que isso aconteça, podemos utilizar o **gerenciamento de configuração** do Puppet, que consegue manter a máquina no estado padrão sem que ninguém precise reexecutar o arquivo de `manifest`. O Puppet faz essa verificação de configuração com intervalo customizável, chamamos isso de **self-healing**.


Nesta aula, aprendemos que:

-   O Ansible, assim como o Puppet, é uma ferramenta para provisionar uma máquina
-   O Ansible envia comandos SSH para a máquina a ser configurada, e não precisa de um cliente instalado (apenas o Python)
-   Os passos de instalação são configurados de alto nível, dentro de um _playbook_.
-   O arquivo de inventário (**hosts**) define os alvos da instalação
-   O Vagrant integra o Ansible e tem uma configuração dedicada:
    
    ```
    config.vm.provision "ansible" do |ansible|
     ansible.inventory_path = "hosts"
     ansible.playbook = "playbook.yml"
    end
    ```

