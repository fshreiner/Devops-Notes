Virtualização VS Container


O que são Containers?
Resumidamente, aprendemos a controlar o VirtualBox a partir do Vagrant estabelecendo nossas regras em um arquivo de texto para configurar diferentes ferramentas em múltiplas máquinas virtuais, a fim de gerar infraestruturas de código eficientes e otimizar o trabalho no computador de desenvolvedores.

Agora, abordaremos a função de **containers** que possuem o mesmo objetivo do esquema de virtualização que vimos neste curso e ainda são mais leves; isolar o MySQL separado do sistema operacional, possibilitando a instalação de várias versões sem conflitos entre si.

Ao invés de trabalharmos com um **hipervisor**, utilizamos o recurso de **Container Engine** representando pelo **Docker**, o qual depende do Linux. Logo, não temos simulações de outros sistemas operacionais para rodar as ferramentas.

Ou seja, se já temos uma biblioteca de dados carregada no sistema operacional, não é preciso que os programas configurados possuam suas própria, pois se conecta diretamente à fonte através dos containers, tornando-os mais leves e rápidos.

Esta técnica se tornou bastante popular justamente por sua eficiência e praticidade, mas tudo depende do padrão da empresa como critério de escolha ou necessidade específica.

No caso do Windows, há poucas versões que comportam os containers automaticamente, tornando pouco acessível. Ainda, se formos até a documentação do Docker [nesta página](http://docs.docker.com/docker-for-windows/install/), encontramos o link para baixar do **Docker Hub**, e nos detalhes encontramos a informação que há necessidade de ter instalado o **Hyper-V**, o qual não é compatível com todas as versões Windows.

Por isso, podemos simular o Linux através do Vagrant para instalar o programa de container. Faremos isso no próximo passo.



Docker no Windows?
Criaremos um ambiente com Vagrant para provisionar o Docker no Windows.

No "Vagrantfile", adicione ao final do texto:

```
    config.vm.define "dockerhost" do |dockerhost|
        dockerhost.vm.provider "virtualbox" do |vb|
            vb.memory = 512
            vb.cpus = 1
            vb.name = "ubuntu_dockerhost"
        end

        dockerhost.vm.provision "shell", 
            inline: "apt-get update && apt-get install -y docker.io"
    end
```

No terminal, escreva `vagrant up dockerhost` e aguarde o fim da operação.

Conecte-se com `vagrant ssh dockerhost` e verifique a versão digitando `sudo docker --version`. Desta forma, já temos um ambiente de estudos para trabalharmos.

Se quisermos instalar um box com configurações específicas, no **Vagrant Cloud** é possível fazer busca por estes de acordo com sua necessidade, sem que precisemos realizar toda a estruturação do zero.

Para subir um container com uma imagem específica, insira `sudo docker run hello-world`. Repare que a saída aponta que o programa não identificou este pacote no host e automaticamente baixou da fonte, dentre outras mensagens e comandos relevantes.



Conclusão
Nesta aula, aprendemos que:

-   O Docker é uma tecnologia para criar, rodar e administrar _containers_, baseado no Linux
-   _Containers_ virtualizam o sistema operacional
-   Máquinas virtuais virtualizam o hardware
-   _Containers_ são mais leves do que máquinas virtuais
-   Ambos, _containers_ e máquinas virtuais, servem para rodar e isolar processos e aplicações


**Parabéns!** Chegamos ao final do curso de **Vagrant: gerenciando máquinas virtuais**.

Relembraremos nossos passos: começamos pela configuração simples de uma máquina virtual para podermos estabelecer as diretrizes de rede com chaves públicas e privadas, bem como o processo de port forwarding.

Vimos vários detalhes sobre a conexão SSH gerando suas próprias chaves, pastas compartilhadas, configurações de memória, instalação de pacotes através dos provisioners como Shell, Puppet e Ansible.

Aprendemos sobre comandos importantes do Vagrant, comparamos sua atividade com o Docker e como este funciona, vendo inclusive sua atuação pelo Windows.

Ou seja, a preparação para trabalhar com esta ferramenta é suficiente para permitir novos projetos.

Agradecemos e recomendamos os cursos da **Plataforma Alura** que abordam os recursos apresentados aqui. **Até a próxima!**.


