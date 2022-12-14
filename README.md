# Prova de Conceito


## Descrição
Esta prova de conceito busca verificar o funcionamento de modificações de nomes de domínio em servidores DNS (Domain Name System) e os seus efeitos. Para tanto, foi elaborada uma infraestrutura de servidores independentes da internet, de modo a possibilitar as modificações sem influência externa. Ressalta-se que o atraso nas propagações das atualizações de nomes de domínios não foram levadas em consideração para este experimento, uma vez que estes atrasos dependem de condições de propagação de sinais em redes físicas da internet, o que não poderia ser simulado com exatidão no ambiente virtual.


## Infraestrutura
A infraestrutura criada é composta por 6 (seis) contêineres, sendo eles:
  * Um servidor de nomes com sistema operacional Ubuntu 22.04 LTS e o serviço BIND9 na versão 9.18-22.04_beta, oriundo de https://hub.docker.com/r/ubuntu/bind9
  * Dois servidores web com sistema operacional Debian 11 e o serviço Apache2, oriundo de https://hub.docker.com/_/httpd
  * Dois servidores de e-mail com sistema operacional Ubuntu 22.04.1 LTS, com o serviço Postfix para o protocolo SMTP e o Dovecot para IMAP e POP3
  * Um contêiner com um servidor SSH, um cliente para navegação web por linha de comando chamado Lynx e um cliente de email (SMTP, IMAP e POP3) por linha de comando chamado Mutt.
  
Ressalta-se que todos os arquivos de configuração necessários para o funcionamento dos servidores encontra-se na pasta volumes deste repositório. Do mesmo jeito, o arquivo docker-compose.yml já realiza o mapeamento automático dos arquivos de configuração citados.


## Infraestrutura e Testes
Para iniciar a infraestrutura da prova de conceito, precisa-ser ter instalado no computador os programas Docker e Docker Compose. Após isso, basta executar o comando "docker-compose up" que será levantada toda a infraestrutura necessária. Ressalta-se que o servidor Dovecot dos serviores de e-mail não estão iniciando por padrão junto com aqueles contêineres. Nesse caso, para iniciá-los, basta executar os comandos "docker exec OriginalMailServer dovecot" e "docker exec AttackerMailServer dovecot". Os testes podem ser realizados com uma conexão ssh para a porta 22 da máquina local, uma vez que o cliente será mapeado para esta localização.
Uma vez dentro da máquina cliente, basta utilizar os comandos:
  * lynx www.poc.exemplo
  * mutt

O primeiro comando irá realizar a resolução do nome "www.poc.exemplo" e em seguida acessar a página no respectivo servidor web. O segundo comando irá realizar a resolução do nome "mail.poc.exemplo" e em seguida tentar uma conexão IMAP com o servidor de e-mail. Caso a conexão seja recusada, pode ser que tenha se esquecido de iniciar o servidor Dovecot dentro do servidor de e-mail, nesse caso verifique o início desta Seção.


## Cenário de Ataque
Para realizar os testes do cenário de ataque deve-se seguir a seguinte sequência de eventos:
  * Levantar a infraestrutura necessária, lembrando de iniciar o serviço Dovecot nos servidores de e-mail
  * Acessar o cliente por ssh na porta 22 da máquina local (ssh perito@127.0.0.1), a senha é 12345
  * A partir do cliente, acessar o servidor web (lynx www.poc.exemplo) e verificar se foi acessada a página correta
  * A partir do cliente, acessar o servidor de e-mail com o comando mutt
  * Enviar alguns emails entre as contas user1@poc.exemplo e user2@poc.exemplo
  * Verificar se os mesmos constam na caixa de entrada e saída do respectivo usuário
  * Modificar os arquivos db.poc.exemplo e db.rev.poc.exemplo localizados em volumes/dns_server/bind trocando os endereços IP do servidor web e de e-mail para os números dos atacantes
  * Reiniciar o servidor de nomes com docker exec DNSServer service named restart
  * Realizar novamente os acessos ao servidor web e de email
  * Enviar novos emails para as contas user1@poc.exemplo e user2@poc.exemplo
  * Verificar o que foi acessado no servidor web e quais e-mails se tem acesso no servidor de e-mail
  * Retornar os valores corretos dos endereços IP dos servidores web e de e-mail
  * Verificar novamente o que se pode ser acessado nos dois serviços.


## Resultados Obtidos
Mediante as análises das simulações realizadas na infraestrutura do POC criado, foi possível extrair os seguintes resultados que seguem formatados em tópicos para uma consulta mais rápida.
  * A modificação no registro de domínio (DNS) causa o redirecionamento das conexões destinadas ao servidor web da empresa. Como consequência, usuários que queiram acessar a página da empresa serão redirecionados para uma página falsa ou receberão um erro. Isso pode causar a impressão de que a página da empresa encontra-se fora do ar para o usuário comum. No entanto, tecnicamente apenas o seu computador não está encontrando o servidor correto.
  * Após o retorno do registro de nome DNS correto e o tempo de propagação das atualizações na hierarquia dos catálogos de nomes, o serviço web retoma a sua normalidade completa.
  * No que tange servidores de e-mail, a modificação no registro de nomes DNS causa o redirecionamento de conexões vindas de clientes e de outros servidores de e-mail. Neste caso, se o atacante possuir um servidor de e-mail especialmente configurado com as mesmas contas de usuários do servidor original porém com senhas diferentes, terá acesso a todos os e-mails enviados para estes a partir da modificação do registro de nomes. Do mesmo modo, poderá enviar e-mails a partir das contas de usuários se passando pelos originais, sem causar desconfiança nos destinatários.
  * Ressalta-se que o atacante não terá acesso aos e-mails recebidos pelas contas antes da modificação no registro de nomes. Do mesmo modo, quando do retorno do registro à normalidade, o atacante também não terá acesso aos futuros e-mails enviados para as contas.
  * Os usuários originais das contas de e-mails não terão acesso aos e-mails recebidos durante a alteração do registro de nomes, uma vez que estes foram entregues ao servidor de e-mails do atacante e as senhas são diferentes. A tentativa de acesso dos usuários originais às suas contas de e-mail durante o ataque resultará em uma mensagem de senha inválida.
  * Após o retorno da normalidade do registro de nomes, o servidor de e-mails e seus usuários retornam à completa normalidade, percebendo apenas um lapso sem a recepção de e-mails e reclamações de destinatários quanto a possíveis e-mails fraudulentos enviados a partir de suas contas.
