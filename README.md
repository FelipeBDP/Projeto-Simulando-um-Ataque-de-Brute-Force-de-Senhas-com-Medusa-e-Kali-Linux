# Relatório da simulação de Ataque de Brute Force de Senhas com Medusa e Kali Linux

 O relatório será dividido em 3 partes. Na primeira parte irei passar pelo hardware e software usado na simulação. Na segunda parte irei explicar como foi feito a simulação do ataque de Brute Force e na última eatapa, irei comentar quais ações podem ser usadas para mitigar as vunerabilidades exploradas.

## Hardware e Software

 Utilizei um notebook com 32GB de mémoria RAM e um processador AMD Ryzen 7 5800H com 8 núcleos e 16 threads. Na parte de software, usei o VirtualBox para virtualizar os sistemas operacionais Kali Linux e um linux modificado para pentest chamado Metasploitable. Já os softwares para executar as simulações foram: Enum4linux, NMap, Medusa e Hydra.

## Executando as simulações de Brute Force

### Encontrando o alvo

No inicio da simulação, é aconselhado entrar na máquina virtual onde está sendo executado o Metasploitable e executar o comando ip -a para identificar o ip para usarmos na simulação. Devido a um problema na configuração da placa de rede no VirtuaBox, decidi usar um roteador de backup e sem internet para fazer a simulação. Além disso coloquei as placas de rede em modo Bridge e o problema foi resolvido.

Como fiz essa alteração na rede que as máquinas virtuais estavam conectadas, decidi usar o comando nmap -sS endereço de rede/24 para executar uma varredura na rede e descobrir quais máquinas estavam conectadas no roteador. É importante salientar que coloquei mais algumas dispositivos na rede de proposito para a simulação ficar mais realista.

Sobre o comando nmap -sS endereço de rede/24, o -sS envia pacotes TCP com a flag SYN para as portas do alvo com intuito de iniciar uma conexão. Ao receber a resposta para continuar a conexão, o nmap não continua. Esse método é usado por ser um furtivo. Já a questão do endereço de rede/24, é para que o nmap faça a varredura da rede inteira. Caso fosse necessário enviar direto ao alvo, era só susbtituir o endereço de ip da rede pelo endereço de ip do alvo sem o /24.

Abaixo segue o prints da tela com a saída do comando:

![nome-da-imagem1](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_01_06.png?raw=true)
![nome-da-imagem2](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_01_13.png?raw=true)

### Verificando quais portas estão abertas

Após identificar o alvo, o passo seguinte foi verificar se as portas ftp, ssh, http e smdb estavam abertas. Para fazer essa tarefa foi executado o comando: nmap -sV -p 21,22,80,445,139 endereço de ip

Este comando escaneia as portas 21(ftp) ,22(ssh) ,80(http) ,445(smdb) e 139(smdb). O parâmetro -sV identifica a versão do serviço que está rodando em cada porta.

Abaixo segue o print da tela com a saída do comando:

![nome-da-imagem3](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_01_20.png?raw=true)

### Verificando se a porta FTP está ativa

Na simulação, o primeiro ataque seria na porta ftp. Para verificar se a porta ftp estava ativa, foi usado o comando: ftp endereço de ip. Ao executar o comando, tivemos o retorno pedindo um usuário. Esse retorno indica que o ftp está ativo. É importante observar que só temos um endereço de ip como alvo e sabemos que a porta ftp está aberta, mas não conseguimos acessar nada somente com essas informações.

Abaixo segue o print da tela com a saída do comando:

![nome-da-imagem4](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/ftp-teste.png?raw=true)

### Preparando o ataque Brute Force na porta FTP

Para iniciar o ataque de Brute Force, foram criados 2 arquivos de texto com algumas palavras. O primeiro arquivo se chama users.txt e representa uma lista de usuários e o segundo arquivo se chama pass.txt e representa uma lista de senhas. Para criar esses arquivos, foram usados o comandos echo -e "user\nmsfadmin\nadmin\nroot" > users.txt e  echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt, onde o primro comando cria o arquivo de usuários e o segundo comando cria o arquivo de senhas.

O comando echo é usado para mostrar o resultado de um script na tela, mas em nosso exemplo, estamos usando ele para construir um arquivo de texto. O -e é para identificar algum comando especifico como o caso da \. A \ indica que cada palavra ficara em uma linha. Por último temos o >, que indica aonde o texto será salvo.

## Executando o ataque com a Medusa
Com as listas de usuários e senhas prontas, está na hora de executar o ataque de Brute Force. Na simulação que estamos fazendo, o software utilizado é o Medusa. O Medusa irá usar as duas listas para descobrir qual é o usuário e senha que podem acessar a porta FTP do ip 192.168.99.4. O comando utilizado foi medusa -h endereço de ip -U users.txt -P pass.txt -M ftp -t 6.

O comando medusa usa o -h para indicar o endereço ip do alvo; -U é usado para lista de usuários; -P é usado para uma lista de senhas; -t é usado para configurar o número de threads simultâneas, o que torna o ataque mais rápido.

O resultado final do ataque identificou o usuário msfadmin e a senha msfadmin como credenciais válidas.

![nome-da-imagem5]()

### Testando o resultado encontrado

Usando novamente o comando ftp endereço de ip, iremos digitar o usuário msfadmin e a senha msfadmin para verificar se realmente conseguiremos acessar a porta FTP.

Conforme pode ser visto na imagem abaixo, consegumos acessar acesso a porta FTP com sucesso.

![nome-da-imagem6](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/ftp-sucesso.png?raw=true)

### Simulando um Ataque Web(HTTP)

Agora iremos realizar um ataque na porta 80(HTTP). O alvo continua o mesmo, mas nesse ataque, estamos simulando que o nosso alvo tem hospedado um site que exige login e senha. O primeiro passo do nosso ataque foi acessa o site pelo navegador Firefox instalado no kali Linux. O endereço do site é endereço ip do alvo/dvwa/login.php. Antes de digitarmos qualquer usuário e senha, foi pedido que fosse aberto o painel de ferramentas do desenvolvedor na página. Para realizar essa tarefa, foi necessário clicar na tecla f12 e em seguida clicar na guia network, na navegação do tipo POST e em Request, que nos mostrará tudo o que o navegador está enviando e recebendo durante a interação, incluindo os nomes dos parâmetros que o servidor espera receber.

### Executando o ataque na página Web

Para esse ataque, usaremos nvamente o Medusa, mas nesse cas em especifico, o comando ganhará novos parâmetros. Segue o comando utilizado:

medusa -h endereço de ip -U users.txt -P pass.txt -M http -m PAGE:'/dvwa/login.php' -m FORM:'username=^USER^&password=^PASS^&Login=Login' -m 'FAIL=Login failed' -t 6

A difrença d comando atual para o comando utilizado no ataque a porta FTP, é que temos o uso do -m. Estamos usando o -m para passar alguns parâmetros para o Medusa. O prâmetro PAGE é o caminho da página. Por mais que o Medusa tenha o endereço ip de ataque, o Medusa precisa saber qual é o caminho da página de login onde ocorrerá o ataque de brute force. Além disso, precisamos indicar os nomes dos campos que estão na página de login, por isso temos o parâmetro FORM e os nomes username e password. Se a página usasse outros nomes como login e pass, teríamos que realizar essa alteração. O parâmetro FAIL, é usado para idnetificar quais usuários e senhas da nossa lista retornaram com falha.

Segue resultado do comando:

![nome-da-imagem7](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_02_02.png?raw=true)

### Um resultado não esperado

Seguindo os passos da instrutora percebei que tinha algo errado. Como pode ser notado na imagem anterior, todos os usuários e senhas retornaram sucesso. Mas somente o usuário admin e a senha password eram as credenciais corretas. Isso ficou evidnete quando a instrutora tentou usar outras credenciais para acessa a página que tinhamos acabado de atacar. Isso me fez questionar se o Medusa tinha problema com falso positivo. O falso positivo acontece quando um resposta falsa é classificada como correta ou o inverso acontece. Em nosso exemplo tinhamos 16 combinações mais somente 1 era a correta. Para verificar se o problema era do Medusa, decidi usar um outro software que tinha sido abordado na parte introdutório da simulação. O software em questão é o Hydra. O Hydra também permite fazer ataque de brute force como o Medusa.

### Refazendo o ataque com o Hydra

Para realizar o ataque com o Hydra, usei  comando: hydra -L users.txt -P pass.txt endereço ip -V http-form-post '/dvwa/login.php:Username=^USER^Password=^PASS^&wp-submit=Log Int&testcookie=1:Login failed'.

Para que o hydra utilizasse as listas que já tinha criado, usei os parâmetros -L para ler a lista de usuários e -P para ler a lista de senhas; Em seguida passamos o endereço ip do alvo; o parâmetro -V indica que o ataque será feito a um formulário http usando o método Post e entre aspas simples, temos o caminho da página de login, os nomes dos campos na página de login e com será feita a captura de falha ao usarmos nossas listas.

Abaixo temos o resultado do comando exectado:

![nome-da-imagem8](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_02_12.png?raw=true)

### Possivel problema

Avaliando o resultado do Hydra, percebi que tive o mesmo resultado que encontrei no Medusa. Como ambas as ferramentas tiveram uma alta taxa de erro, decidi procurar algo sobre esse problema. Durante as buscas na internet, identifiquei alguns relatos sobre um problema no retorno da página atacada para ambas as ferramentas. Isso significa que quando a página ataca não retorna o resultado que aguardamos de forma exata, tanto o Hydra como o Medusa geram falso positivo. Isso indica que a página que estamos usando na simulção está trazendo várias informações na resposta do nosso ataque e os ambos os softwares estão com dificuldade de filtrar o resultado.

### Simulando um Ataque a porta SMB (Server Message Block)

Na úlitma simulação foi um ataque a porta SMB. Antes de realizar o ataque, foi utilizado o comando enum4linux -a endereço ip | tee enum4_output.txt. Esse comando gerou um arquivo com várias informações sobre a máquina que está sendo atacada. Dentre as informações, temos os usuários, pastas e outras informações.

A seguir temos os prints de todas as informações que conseguimos captura:

![nome-da-imagem9](https://github.com/Fe

![nome-da-imagem10](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_02_26.png?raw=true)

![nome-da-imagem11](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_02_35.png?raw=true)

![nome-da-imagem12](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_02_53.png?raw=true)

![nome-da-imagem13](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_03_01.png?raw=true)

![nome-da-imagem14](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_03_04.png?raw=true)

3º - Criando wordlists de usuários

No comando anterior conseguimos acesso aos nomes dos usuários reais, agora precisamos criar nosso arquivo de alvos e nosso arquivo de senhas.

Comando: echo -e "user\nmsfadmin\nservice" > smb_users.txt

Comando: echo -e "password\n123456\nWelcome123\nmsfadmin" > senhas_spray

Ao contrário do brute force, que testa muitas senhas em um único usuário, o password spraying testa poucas senhas em muitos usuários.

![nome-da-imagem15](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Captura%20de%20tela%202026-03-05%20153806.png?raw=true)

4º - Rodando ataque com Medusa

Comando: medusa -h nú.me.ro.ip -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50

Onde: 

-h é o IP do nosso alvo. 

-U é a lista de usuários descoberta na enumeração. 

-P é a lista de senhas fracas. 

-M smbnt é o módulo específico para ataques via smb. 

-t 2 é uma das duas threads simultâneas, que simulam 2 usuários testando senhas. 

-T 50 significa até 50 hosts em paralelo.

![nome-da-imagem16](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Captura%20de%20tela%202026-03-05%20153815.png?raw=true)

5º - Testando o acesso utilizando smbclient

Verifica se teremos acesso como administrador através das credenciais encontradas no ataque anterior.

Comando: smbclient -L //192.168.56.101 -U msfadm

![nome-da-imagem17](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_03_30.png?raw=true)
