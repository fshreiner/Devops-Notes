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


