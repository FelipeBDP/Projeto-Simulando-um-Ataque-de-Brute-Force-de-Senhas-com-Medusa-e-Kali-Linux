# Relatório da simulação de Ataque de Brute Force de Senhas com Medusa e Kali Linux

 O realtório será dividido em 3 partes. Na primeira parte irei passar pelo hardware e software usado na simulação. Na segunda parte irei explicar como foi feito a simulação do ataque de Brute Force e na última eatapa, irei comentar quais ações podem ser usadas para mitigar as vunerabilidades exploradas.

## Hardware e Software

 Utilizei um notebook com 32GB de mémoria RAM e um processador AMD Ryzen 7 5800H com 8 núcleos e 16 threads. Na parte de software, usei o VirtualBox para virtualizar os sistemas operacionais Kali Linux e um linux modificado para pentest chamado Metasploitable. Já os softwares para executar as simulações foram: NMap, Medusa e Hydra.

## Executando a simulação de Brute Force

### Encontrando o alvo

No inicio da simulação, é aconselhado entrar na máquina virtual onde está sendo executado o Metasploitable e executar o comando ip -a para identificar o ip para usarmos na simulação. Devido a um problema na configuração da placa de rede no VirtuaBox, decidi usar um roteador de backup e sem internet para fazer a simulação. Além disso coloquei as placas de rede em modo Bridge e o problema foi resolvido.

Como fiz essa alteração na rede que as máquinas virtuais estavam conectadas, decidi usar o comando nmap -sS endereço de rede/24 para executar uma varredura na rede e descobrir quais máquinas estavam conectadas no roteador. É importante salientar que coloquei mais algumas dispositivos na rede de proposito para a simulação ficar mais realista.

Sobre o comando nmap -sS endereço de rede/24, o -sS ==> envia pacotes TCP com a flag SYN para as portas do alvo com intuito de iniciar uma conexão. Ao receber a resposta para continuar a conexão, o nmap não continua. Esse método é usado por ser um furtivo. Já a questão do endereço de rede/24, é para que o nmap faaça a varredura da rede inteira. Caso fosse necessário enviar diferto ao alvo, era só susbtituir o endereço de ip da rede pelo endereço de ip do alvo sem o /24.

Abaixo segue o prints da tela com a sáida do comando:

![nome-da-imagem](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_01_06.png?raw=true)
![nome-da-imagem1](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_01_13.png?raw=true)

### Verificando quais portas estão abertas

Após identificar o alvo, o passo seguinte foi verificar se as portas ftp, ssh, http e smdb estavam abertas. Para fazer essa tarefa foi executado o comando: nmap -sV -p 21,22,80,445,139 endereço de ip

Este comando escaneia as portas 21(ftp) ,22(ssh) ,80(http) ,445(smdb) e 139(smdb). O parâmetro -sV identifica a versão do serviço que está rodando em cada porta.

Abaixo segue o print da tela com a sáida do comando:

![nome-da-imagem2](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_01_20.png?raw=true)

### Verificando se a porta FTP está ativa

Na simulação, o primeiro ataque seria na porta ftp. Para verificar se a porta ftp estava ativa, foi usado o comando: ftp endereço de ip. Ao executar o comando, tivemos o retorno pedindo um usuário. Esse retorno indica que o ftp está ativo. É importante observar que só temos um endereço de ip como alvo e sabemos que a porta ftp está aberta, mas não conseguimos acessar nada somente com essas informações.

Abaixo segue o print da tela com a sáida do comando:

![nome-da-imagem3](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_01_28.png?raw=true)

### Preparando o ataque Brute Force

Para iniciar o ataque de Brute Force, foram criados 2 arquivos de texto com algumas palavras. O primeiro arquivo se chama users.txt e representa uma lista de usuários e o segundo arquivo se chama pass.txt e representa uma lista de senhas. Para criar esses arquivos, foram usados o comandos echo -e "user\nmsfadmin\nadmin\nroot" > users.txt e  echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt, onde o primro comando cria o arquivo de usuários e o segundo comando cria o arquivo de senhas.

O comando echo é usado para mostrar o resultado de um script na tela, mas em nosso exemplo, estamos usando ele para construir um arquivo de texto. O -e é para identificar algum comando especifico como o caso da \. A \ indica que cada palavra ficara em uma linha. Por último temos o >, que indica aonde o texto será salvo.

## Executando o ataque com a Medusa
Com as listas de usuários e senhas prontas, está na hora de executar o ataque de Brute Force. Na simulação que estamos fazendo, o software utilizado é o Medusa. O Medusa irá usar as duas listas para descobrir qual é o usuário e senha que podem acessar a porta FTP do ip 192.168.99.4. O comando utilizado foi medusa -h endereço de ip -U users.txt -P pass.txt -M ftp -t 6.

O comando medusa usa o -h para indicar o endereço ip do alvo; -U é usado para lista de usuários; -P é usado para uma lista de senhas; -t é usado para configurar o número de threads simultâneas, o que torna o ataque mais rápido.

O resultado final do ataque identificou o usuário msfadmin e a senha msfadmin como credenciais válidas.

## testando o resultado encontrado

Usando novamente o comando ftp endereço de ip, iremos digitar o usuário msfadmin e a senha msfadmin para verificar se realmente conseguiremos acessar a porta FTP.

Conforme pode ser visto na imagem abaixo, consegumos acessar acesso a porta FTP com sucesso.


### Simulando Ataque web (http) Simulando ataque brute force em formulários de login web (http) no sistema dvwa

1º - Acessar, no navegador firefox do Kali Linux, o endereço nú.me.ro.ip/dvwa/login.php para visualizar a página de teste de login do dvwa.

Na sequência abrir o painel de ferramentas do desenvolvedor na página de teste de login do dvwa clicando em f12 e em seguida clicar na guia network, na navegação do tipo POST e em Request, que nos mostrará tudo o que o navegador está enviando e recebendo durante a interação, incluindo os nomes dos parâmetros que o servidor espera receber. A Medusa vai simular em cima destes parâmetros.


2º - No terminal do Kali, após criadas as wordlists de usuários e de senhas, rodar o seguinte comando com a Medusa.

Comando: medusa -h nú.me.ro.ip -U users.txt -P pass.txt -M http

-m PAGE:'/dvwa/login.php'

-m FORM:'username=^USER^&password=^PASS^&Login=Login'

-m 'FAIL=Login failed' -t 6

As credenciais corretas encontradas aparecerão com a palavra SUCCESS.

Em seguida utilizamos o primeiro login e senha encontrados para acessar o sistema.


### Simulando Ataque SMB (Server Message Block) Simulando ataques de enumeração e spraying contra o serviço SMB (Server Message Block)

1º - Rodar a enumeração de usuários com enum4linux

Comando: enum4linux -a 192.168.56.101 | tee enum4_output.txt

2º - Na sequência podemos abrir o arquivo do comando que acabamos de rodar e visualizar usuários que sejam possíveis alvos de ataques. O número rid é o identificador relativo do usuário no sistema. Sempre que houver nomes de usuários genéricos como null ou interrogação geralmente são de usuários mais vulneráveis.

Comando: less enum4_output.txt

3º - Criando wordlists de usuários

No comando anterior conseguimos acesso aos nomes dos usuários reais, agora precisamos criar nosso arquivo de alvos e nosso arquivo de senhas.

Comando: echo -e "user\nmsfadmin\nservice" > smb_users.txt

Comando: echo -e "password\n123456\nWelcome123\nmsfadmin" > senhas_spray

Ao contrário do brute force, que testa muitas senhas em um único usuário, o password spraying testa poucas senhas em muitos usuários.


4º - Rodando ataque com Medusa

Comando: medusa -h nú.me.ro.ip -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50

Onde: 

-h é o IP do nosso alvo. 

-U é a lista de usuários descoberta na enumeração. 

-P é a lista de senhas fracas. 

-M smbnt é o módulo específico para ataques via smb. 

-t 2 é uma das duas threads simultâneas, que simulam 2 usuários testando senhas. 

-T 50 significa até 50 hosts em paralelo.


5º - Testando o acesso utilizando smbclient

Verifica se teremos acesso como administrador através das credenciais encontradas no ataque anterior.

Comando: smbclient -L //192.168.56.101 -U msfadm
