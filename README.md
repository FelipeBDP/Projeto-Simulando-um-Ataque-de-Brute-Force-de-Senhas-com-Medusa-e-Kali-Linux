# Relatório da simulação de Ataque de Brute Force de Senhas com Medusa e Kali Linux

O relatório será dividido em 3 partes. Na primeira parte, passarei pelo hardware e software usados na simulação. Na segunda parte, explicarei como foi feita a simulação do ataque de Brute Force e, na última etapa, comentarei quais ações podem ser usadas para mitigar as vulnerabilidades exploradas.

## Hardware e Software

Utilizei um notebook com 32 GB de memória RAM e um processador AMD Ryzen 7 5800H com 8 núcleos e 16 threads. Na parte de software, usei o VirtualBox para virtualizar os sistemas operacionais Kali Linux e um Linux modificado para pentest chamado Metasploitable. Já os softwares para executar as simulações foram: Enum4linux, NMap, Medusa e Hydra.

## Executando as simulações de Brute Force

### Encontrando o alvo

No início da simulação, é aconselhado entrar na máquina virtual onde está sendo executado o Metasploitable e executar o comando ip -a para identificar o IP a ser usado na simulação. Devido a um problema na configuração da placa de rede no VirtualBox, decidi usar um roteador de backup sem internet para fazer a simulação. Além disso, coloquei as placas de rede em modo Bridge e o problema foi resolvido.

Como fiz essa alteração na rede em que as máquinas virtuais estavam conectadas, decidi usar o comando nmap -sS endereço de rede/24 para executar uma varredura na rede e descobrir quais máquinas estavam conectadas ao roteador. É importante salientar que coloquei mais alguns dispositivos na rede de propósito para a simulação ficar mais realista.

Sobre o comando nmap -sS endereço de rede/24, o parâmetro -sS envia pacotes TCP com a flag SYN para as portas do alvo com o intuito de iniciar uma conexão. Ao receber a resposta para continuar a conexão, o NMap não prossegue. Esse método é usado por ser furtivo. Já a questão do endereço de rede/24 é para que o NMap faça a varredura da rede inteira. Caso fosse necessário enviar direto ao alvo, bastaria substituir o endereço de IP da rede pelo endereço de IP do alvo sem o /24.
Abaixo segue o print da tela com a saída do comando:

![nome-da-imagem1](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_01_06.png?raw=true)
![nome-da-imagem2](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_01_13.png?raw=true)

### Verificando quais portas estão abertas

Após identificar o alvo, o passo seguinte foi verificar se as portas FTP, SSH, HTTP e SMB estavam abertas. Para fazer essa tarefa, foi executado o comando:
nmap -sV -p 21,22,80,445,139 endereço de ip

Este comando escaneia as portas 21 (FTP), 22 (SSH), 80 (HTTP), 445 (SMB) e 139 (SMB). O parâmetro -sV identifica a versão do serviço que está rodando em cada porta.

Abaixo segue o print da tela com a saída do comando:

![nome-da-imagem3](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_01_20.png?raw=true)

### Verificando se a porta FTP está ativa

Na simulação, o primeiro ataque seria na porta ftp. Para verificar se a porta ftp estava ativa, foi usado o comando: ftp endereço de ip. Ao executar o comando, tivemos o retorno pedindo um usuário. Esse retorno indica que o ftp está ativo. É importante observar que só temos um endereço de ip como alvo e sabemos que a porta ftp está aberta, mas não conseguimos acessar nada somente com essas informações.

Abaixo segue o print da tela com a saída do comando:

![nome-da-imagem4](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/ftp-teste.png?raw=true)

### Preparando o ataque Brute Force na porta FTP

Para iniciar o ataque de Brute Force, foram criados 2 arquivos de texto com algumas palavras. O primeiro arquivo se chama users.txt e representa uma lista de usuários e o segundo arquivo se chama pass.txt e representa uma lista de senhas. Para criar esses arquivos, foram usados os comandos echo -e "user\nmsfadmin\nadmin\nroot" > users.txt e  echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt, onde o primeiro comando cria o arquivo de usuários e o segundo comando cria o arquivo de senhas.

O comando echo é usado para mostrar o resultado de um script na tela, mas em nosso exemplo, estamos usando ele para construir um arquivo de texto. O -e é para identificar algum comando especifico. A \ indica que cada palavra ficara em uma linha. Por último temos o >, que indica aonde o texto será salvo.

## Executando o ataque com a Medusa

Com as listas de usuários e senhas prontas, está na hora de executar o ataque de Brute Force. Na simulação que estamos fazendo, o software utilizado é o Medusa. O Medusa usará as duas listas para descobrir qual é o usuário e senha que podem acessar a porta FTP do ip 192.168.99.4. O comando utilizado foi medusa -h endereço de ip -U users.txt -P pass.txt -M ftp -t 6.

O comando medusa usa o -h para indicar o endereço ip do alvo; -U é usado para lista de usuários; -P é usado para uma lista de senhas; -t é usado para configurar o número de threads simultâneas, o que torna o ataque mais rápido.

O resultado final do ataque identificou o usuário msfadmin e a senha msfadmin como credenciais válidas.

### Testando o resultado encontrado

Usando novamente o comando ftp endereço de ip, digitaremos o usuário msfadmin e a senha msfadmin para verificar se realmente conseguiremos acessar a porta FTP.

Conforme pode ser visto na imagem abaixo, conseguimos acessar acesso a porta FTP com sucesso.

![nome-da-imagem6](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/ftp-sucesso.png?raw=true)

### Simulando um Ataque Web(HTTP)

Agora realizaremos um ataque na porta 80(HTTP). O alvo continua o mesmo, mas nesse ataque, estamos simulando que o nosso alvo tem hospedado um site que exige login e senha. O primeiro passo do nosso ataque foi acessar o site pelo navegador Firefox instalado no Kali Linux. O endereço do site é endereço ip do alvo/dvwa/login.php. Antes de digitarmos qualquer usuário e senha, foi pedido que fosse aberto o painel de ferramentas do desenvolvedor na página. Para realizar essa tarefa, foi necessário clicar na tecla f12 e em seguida clicar na guia network, na navegação do tipo POST e em Request, que nos mostrará tudo o que o navegador está enviando e recebendo durante a interação, incluindo os nomes dos parâmetros que o servidor espera receber.

### Executando o ataque na página Web
Para esse ataque, usaremos novamente o Medusa, mas nesse caso em específico, o comando ganhará novos parâmetros.

Segue o comando utilizado: medusa -h endereço de ip -U users.txt -P pass.txt -M http -m PAGE:'/dvwa/login.php' -m FORM:'username=^USER^&password=^PASS^&Login=Login' -m 'FAIL=Login failed' -t 6

A diferença do comando atual para o comando utilizado no ataque a porta FTP, é que temos o uso do -m. Estamos usando o -m para passar alguns parâmetros para o Medusa. O parâmetro PAGE é o caminho da página. Por mais que o Medusa tenha o endereço ip de ataque, o Medusa precisa saber qual é o caminho da página de login onde ocorrerá o ataque de brute force. Além disso, precisamos indicar os nomes dos campos que estão na página de login, por isso temos o parâmetro FORM e os nomes username e password. Se a página usasse outros nomes como login e pass, teríamos que realizar essa alteração. O parâmetro FAIL, é usado para identificar quais usuários e senhas da nossa lista retornaram com falha.

Segue resultado do comando:

![nome-da-imagem7](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_02_02.png?raw=true)

### Um resultado não esperado

Seguindo os passos da instrutora, percebei que tinha algo errado. Como pode ser notado na imagem anterior, todos os usuários e senhas retornaram sucesso. Mas somente o usuário admin e a senha password eram as credenciais corretas. Isso ficou evidente quando a instrutora tentou usar outras credenciais para acessa a página que tínhamos acabado de atacar. Isso me fez questionar se o Medusa tinha problema com falso positivo. O falso positivo acontece quando uma resposta falsa é classificada como correta ou o inverso acontece. Em nosso exemplo tínhamos 16 combinações mais somente 1 era a correta. Para verificar se o problema era do Medusa, decidi usar um outro software que tinha sido abordado na parte introdutório da simulação. O software em questão é o Hydra. O Hydra também permite fazer ataque de brute force como o Medusa.

### Refazendo o ataque com o Hydra

Para realizar o ataque com o Hydra, usei o comando: hydra -L users.txt -P pass.txt endereço ip -V http-form-post '/dvwa/login.php:Username=^USER^Password=^PASS^&wp-submit=Log Int&testcookie=1:Login failed'.

Para que o hydra utilizasse as listas que já tinha criado, usei os parâmetros -L para ler a lista de usuários e -P para ler a lista de senhas; Em seguida passamos o endereço ip do alvo; o parâmetro -V indica que o ataque será feito a um formulário http usando o método Post e entre aspas simples, temos o caminho da página de login, os nomes dos campos na página de login e com será feita a captura de falha ao usarmos nossas listas.

Abaixo temos o resultado do comando executado:

![nome-da-imagem8](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_02_12.png?raw=true)

### Possivel problema

Avaliando o resultado do Hydra, percebi que tive o mesmo resultado que encontrei no Medusa. Como ambas as ferramentas tiveram uma alta taxa de erro, decidi procurar algo sobre esse problema. Durante as buscas na internet, identifiquei alguns relatos sobre um problema no retorno da página atacada para ambas as ferramentas. Isso significa que quando a página atacada não retorna o resultado que aguardamos de forma exata, tanto o Hydra como o Medusa geram falso positivo. Isso indica que a página que estamos usando na simulação está trazendo várias informações na resposta do nosso ataque e os ambos os softwares estão com dificuldade de filtrar o resultado.

### Simulando um Ataque a porta SMB (Server Message Block)

Na última simulação foi um ataque a porta SMB. Antes de realizar o ataque, foi utilizado o comando enum4linux -a endereço ip | tee enum4_output.txt. Esse comando gerou um arquivo com várias informações sobre a máquina que está sendo atacada. Dentre as informações, temos os usuários, pastas e outras informações.

A seguir temos os prints de todas as informações que foram captura:

![nome-da-imagem9](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_02_20.png?raw=true)

![nome-da-imagem10](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_02_26.png?raw=true)

![nome-da-imagem11](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_02_35.png?raw=true)

![nome-da-imagem12](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_02_53.png?raw=true)

![nome-da-imagem13](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_03_01.png?raw=true)

![nome-da-imagem14](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_03_04.png?raw=true)

### Criando os arquivos de usuário e lista para o ataque

Como visto no ataque a porta FTP, repeti o processo para criar uma lista nova de usuários e senhas usando os comandos abaixo:

Comando: echo -e "user\nmsfadmin\nservice" > smb_users.txt

Comando: echo -e "password\n123456\nWelcome123\nmsfadmin" > senhas_spray

![nome-da-imagem15](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Captura%20de%20tela%202026-03-05%20153806.png?raw=true)

### Executando o ataque com Medusa na porta SMB

Ao contrário do primeiro ataque que foi de brute force, nesta simulação foi usada o ataque de password spraying. O ataque password sprayig testa poucas senhas em muitos usuários para evitar que a máquina atacada fique lenta e levante suspeitas sobre o ataque.

O ataque foi feito usando o Medusa e o comando utilizado foi: medusa -h endereço ip -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50

A diferença entre o comando atual e o usado no ataque a porta FTP, está no uso do modulo smbnt que é referente a porta smb; O -t igual a 2 para não causar lentidão na máquina; -T com valor 50 que significa até 50 hosts em paralelo.

Abaixo temos o resultado obtido:

![nome-da-imagem16](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Captura%20de%20tela%202026-03-05%20153815.png?raw=true)

### Testando usuário e senha encontrandos no ataque ao SMB

Com o usuário e senha encontrados na etapa anterior, usamos o comando smbclient -L //endereço ip -U msfadm para acessar a máquina. Conforme pode ser visto na imagem a seguir, conseguimos realizar o acesso com sucesso.

![nome-da-imagem17](https://github.com/FelipeBDP/Projeto-Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux/blob/main/images/Screenshot_2026-03-04_16_03_30.png?raw=true)

## Medidas para Mitigar os problemas encontrados

Conforme visto nas simulações feitas, o uso de senhas fracas, softwares com versões desatualizadas ou mal configuradas, falta de controle na quantidade de vezes que um usuário pode digitar sua senha e até mesmo manter o número padrão para determinadas portas de serviço, são fatores de risco. Todos esses elementos citados podem ser uma porta de entrada para uma invasão, roubo de dados ou um sequestro das informações. Para mitigar os problemas simulados, abaixo seguem algumas sugestões que ajudaram a resolver esses problemas e reduzir o risco de alguns desses ataques.

### Reforço das práticas de autenticação

O ponto de partida para qualquer estratégia de defesa é garantir que o processo de login seja o mais resistente possível.

- Senhas robustas: A exigência de senhas longas e complexas é fundamental. Recomenda-se que tenham pelo menos 14 caracteres, combinando letras maiúsculas e minúsculas, números e símbolos. Além disso, é importante evitar informações óbvias, como datas de aniversário ou nomes de familiares.
 
- Autenticação multifator (MFA): A senha, por si só, não deve ser suficiente. Ao adicionar uma segunda camada de verificação — como um código enviado por SMS ou gerado em aplicativo — o invasor se depara com uma barreira extra, tornando inútil a descoberta da senha.
  
- Controle de tentativas: Configurar limites para tentativas de login é essencial. Após algumas falhas consecutivas, o sistema pode bloquear temporariamente o usuário ou o endereço IP, dificultando ataques automatizados.
  
- CAPTCHA e reCAPTCHA: Esses mecanismos funcionam como filtros contra acessos automatizados, exigindo uma interação humana para prosseguir. Assim, ferramentas de ataque não conseguem avançar sem resolver o desafio.
  
### Proteções na camada de rede

Antes mesmo de chegar ao processo de autenticação, é possível barrar tentativas maliciosas na infraestrutura de rede.

- Firewalls e WAF: Um firewall bem configurado ajuda a identificar e bloquear tráfego suspeito. No caso de servidores web, o uso de um WAF (Web Application Firewall) é especialmente eficaz contra ataques direcionados a aplicações.
  
- Monitoramento de IPs: Observar padrões de acesso é uma prática valiosa. Se um mesmo endereço IP realiza um número exagerado de tentativas de login, ele deve ser bloqueado automaticamente.
 
- Honeypots: Servidores falsos que simulam serviços legítimos podem ser usados como armadilhas. Eles atraem o atacante, registram suas ações e fornecem informações úteis para reforçar a segurança dos sistemas reais.
  
- Alteração de portas padrão: Embora não seja uma solução definitiva, mudar portas de serviços como SSH (por exemplo, da 22 para outra) dificulta ataques automatizados que exploram configurações padrão.
  
### Fortalecimento da administração do sistema

Por fim, a segurança também depende de boas práticas de gestão e manutenção dos sistemas.

- Atualizações constantes: Manter softwares e sistemas operacionais sempre atualizados é crucial para corrigir falhas conhecidas que poderiam ser exploradas.
  
- Gestão de contas: Contas de usuários que não são mais utilizadas devem ser desativadas ou removidas. Isso reduz a superfície de ataque e evita brechas desnecessárias.
  
- Criptografia de dados: Informações sensíveis, como senhas e dados pessoais, precisam estar protegidas por criptografia. Dessa forma, mesmo em caso de invasão, os dados não ficam expostos em texto simples.

Em resumo, a defesa contra ataques de força bruta não depende de uma única solução, mas da combinação de várias práticas que se reforçam mutuamente. É como construir uma muralha: cada camada adicionada aumenta a dificuldade para quem tenta invadir, tornando o sistema mais resiliente e confiável.
