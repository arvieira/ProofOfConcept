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
  * Reiniciar o servidor de nomes com docker exec DNSServer "service named restart"
  * Realizar novamente os acessos ao servidor web e de email
  * Enviar novos emails para as contas user1@poc.exemplo e user2@poc.exemplo
