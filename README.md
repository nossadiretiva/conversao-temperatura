# Aplicação de Conversão de Temperatura

A Conversão de Temperatura é uma aplicação web construída em ***NodeJS*** que realiza a conversão de valores de temperatura de *Celcius para Fahrenheit* e de *Fahrenheit para Celsius*.

## Criando a imagem para executar o container

Foi criado dentro do projeto um arquivo `Dockerfile` contendo o passo-a-passo (receita de bolo) de criação da imagem *Docker* que posteriormente será utilizada para execução da aplicação em um container.

    FROM  node:17.3.1
    WORKDIR  /app
    COPY  ./package*.json  ./
    RUN  npm  install
    COPY  .  .
    EXPOSE  8080
    CMD  [  "node",  "./server.js"  ]

## Ignorando arquivos desnecessários na criação da imagem

No diretório da aplicação podem existir arquivos indesejados e/ou desnecessários que seriam copiados com o comando `COPY` e que poderiam ser ignorados durante o processo de criação da imagem *Docker*.

Para isso foi criado no mesmo diretório onde se encontram os arquivos do projeto o arquivo `.dockerignore` que possui uma lista (com o mesmo padrão do arquivo `.gitignore`) com os diretórios/arquivos que serão ignorados no momento da execução da cópia dos arquivos para dentro da imagem.

## Gerando imagem Docker

Para criação da imagem *Docker* foi executada, dentro do diretório da aplicação, a linha de comando:

    docker build -t nossadiretiva/conversao-temperatura:v1 .

Utilizando o comando `docker build` com o parâmetro `-t` para configurar o nome da imagem a ser criada

## Testando a imagem criada

Para garantir que a imagem foi gerada corretamente foi criado um container local para execução dessa imagem com o seguinte comando:

    docker container run -d --network kubedev_network --name conversao-temperatura -p 8080:8080 nossadiretiva/conversao-temperatura:v1

Utilizando o comando `docker container run` com os seguintes parâmetros:
- `-d` para construir o container em modo *daemon* e não deixar o *prompt* preso durante sua execução
- `--network` para definir uma *network* (previamente criada) onde o container estará conectado a rede  para se comunicar com outros containers
- `--name` para dar um nome ao container
- `-p` para configurar um *bind* de portas entre o *host* (máquina local) e o container, onde a configuração segue o padrão ***"porta acessada pelo host:porta exposta pelo container"***
- e por último o nome completo da imagem a ser utilizada para criar o container

## Subir imagem criada para o *Docker Hub*

Após a criação da imagem precisamos disponibilizá-la para tornar possível a sua utilização de qualquer lugar.

Para isso colocaremos a imagem disponível em um repositório do *Docker Hub* (ferramenta de repositório de imagens *Docker* - https://hub.docker.com/)

Vamos utilizar o comando `docker login` para iniciar uma sessão conectado ao *Docker Hub*, informando os dados de autenticação no *Docker Hub* e com isso conseguimos o seguinte resultado para mostrar que estamos conectados:
> Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: **[usuário para conexão]**
Password: **[senha de autenticação]**
Login Succeeded
> 
>Logging in with your password grants your terminal complete access to your account.
For better security, log in with a limited-privilege personal access token. Learn more at https://docs.docker.com/go/access-tokens/

Após a autenticação no *Docker Hub* vamos realizar o ***push*** da imagem para o repositório do usuário conectado, utilizando o comando:

    docker push nossadiretiva/conversao-temperatura:v1

Como parte das boas práticas de desenvolvimento e manipulação de imagens *Docker* precisamos subir com uma imagem com a *tag* de versionamento ***latest*** para podermos criar um container sem a necessidade de sabermos qual versão de imagem está disponível.

Para isso vamos criar uma imagem com a *tag* de versionamento ***latest*** utilizando o comando:

    docker tag nossadiretiva/conversao-temperatura:v1 nossadiretiva/conversao-temperatura:latest

Logo depois basta apenas realizar o ***push*** desta nova imagem utilizando o mesmo comando utilizado anteriormente apenas alterando a *tag* ***v1*** para ***latest***:

    docker push nossadiretiva/conversao-temperatura:latest

Após executar essa tarefa o *Docker Hub* do usuário conectado deve ficar como mostra a imagem abaixo:

![Docker Hub](https://github.com/nossadiretiva/imagens/blob/master/hub_conversao_temperatura.png?raw=true)

## Realizando teste com a imagem do *Docker Hub*

Para realização dos testes com as imagens que foram disponibilizadas no *Docker Hub*, precisamos primeiro excluir as imagens que estão no *docker* local da máquina e para isso executamos o seguinte comando:

    docker container rm -f conversao-temperatura

> Esse comando é para remover o container gerado com a imagem que queremos testar.

Depois executamos:

    docker image ls

Para visualizarmos as imagens que estão disponíveis no *Docker*.

    REPOSITORY                            TAG            IMAGE ID       CREATED             SIZE
    nossadiretiva/conversao-temperatura   latest         b1eef4b3fda3   About an hour ago   1.02GB
    nossadiretiva/conversao-temperatura   v1             b1eef4b3fda3   About an hour ago   1.02GB

> E antes de iniciar os testes, excluímos as imagens que foram geradas localmente

Para isso executamos os seguintes comandos:

    docker image rm nossadiretiva/conversao-temperatura:v1
    docker image rm nossadiretiva/conversao-temperatura:latest

Após não existir mais vestígios nem das imagens e nem dos containers, executamos o comando, o mesmo executado no teste de criação da imagem, para criarmos um novo container, mas agora com a imagem disponibilizada no *Docker Hub*:

    docker container run -d --network kubedev_network --name conversao-temperatura -p 8080:8080 nossadiretiva/conversao-temperatura:v1
    
Durante a execução do teste foi alcançado o seguinte resultado:

![Página de teste](https://github.com/nossadiretiva/imagens/blob/master/teste_conversao_temperatura.png?raw=true)
