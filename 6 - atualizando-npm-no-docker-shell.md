
# New major version of npm is available! - Atualizando NPM dentro do Docker
</hr>
Pois é desenvolvedor, periodicamente os avanços de uma ferramenta ou tecnologia requerem atualizações e com os gerenciadores de pacote não é diferente, inclusive essa é a mensagem que comumente vemos quando novas features ou correções são diponibilizadas, no caso do NPM. 
</br></br>
Ainda tem dúvida sobre o que é o NPM?</br>
NPM (Node Package Manager) é o gerenciador de pacotes padrão para o ecossistema NodeJS. Ele é uma ferramenta que permite que desenvolvedores instalem, gerenciem e compartilhem pacotes de código JavaScript reutilizáveis. Os pacotes podem conter bibliotecas, frameworks, módulos, ou até mesmo programas inteiros escritos em base JavaScript. Além disso você tem benfcícios como, por exemplo:
<ul>
  <li>Instalação de pacotes externos direto em seu projeto</li>
  <li>Gerenciamento de pacote, o qual o NPM utiliza-se de um auto rastreio para verificar as dependências necessárias em que seu projeto precise para trabalhar</li>
  <li>Criação de scripts personalizados no package.json que permitem a execução de comandos do próprio NPM</li>
</ul>
Porém como é a proposta desse artigo vamos explorar algo mais específico utilizando o Docker no contexto de execução de um container utilizando uma imagem Node.
</br>
<h2>Um pouco sobre o Docker</h2>
O Docker é uma plataforma de código aberto que permite criar, implementar e gerenciar a criação de aplicativo em containers. Um contêiner é uma unidade de software leve que inclui tudo o que é necessário para executar um aplicativo, incluindo o código, runtime, bibliotecas, dependências e variáveis de ambiente. Isso faz o Docker uma ferramenta amplamente utilizada para desenvolver e implantar aplicativos de maneira consistente e escalável, independente de ambiente ou hospedagem, ou seja, com a utilização do Docker não temos problemas como o famoso "na minha máquina funciona...".
</br></br>
<h2>Criando uma imagem Node</h2>
Um dos conceitos fundamentais no Docker é sobre a criação e execução de imagens que são processo criados a partir de camadas, ou seja imagine que temos uma imagem inicial chamada "Scratch", um HD formatado, depois disso precisaremos de partes de um Sistema operacional como o Ubuntu, ovbiamente as partes do ubuntu que precisaremos são apenas as partes que o SO não tem, pois as que ele tem não serão necessárias para baixar.</br>
Agora imaginemos isso no contexto do NodeJS, pois em seu registry (como o DockerHub) no Docker, existem versões que contém partes necessárias para a execução do NodeJS puramente em nossa máquina sem que tenhamos a necessidade de ter o Node instalado localmente, tudo ocorre por meio do Docker.</br></br>
Compreendido esse tópico, vamos criar uma imagem utilizando uma imagem Node para criar a nossa própria.

```bash 
FROM node:18-alpine

WORKDIR /usr/my-app/app

COPY package*.json .

EXPOSE 3000

CMD [ "tail", "-f", "/dev/null" ]
```
</br></br>
<h2>Juntando tudo até aqui</h2>
Compreendido todos esses conceitos até aqui vamos para a nossa execução de fato, o ponto central desse artigo é te mostrar como fazer o update do NPM dentro do Docker toda vez que uma nova versão estiver disponível para ser baixada. Isso pode nos trazer uma série de benefícios como manter nosso software sempre atualizado, evitar incopatibilidades nas versões de recursos a serem utilizados conforme nosso software cresce e mais uma série de benefíicios.
</br></br>
Um outro ponto importante é que ao utilizamos o Node a partir do Docker temos acesso aos recursos do Node em nosso projeto, no caso da utilização do NPM o Node sempre enxerga a versão do que está em nossa máquina em acordo com a versão utilizada na imagem criada para o Docker, e sim, é possível configurar tudo isso. Mas em nosso contexto vamos imaginar que estamos rodando nossa imagem personalidade criada a partir de uma imagem Node no Docker conforme o Dockerfile a cima e para essa execução vamos precisar de mais dois recursos do Docker que são um "docker-compose" e um arquivo em shell chamado "entrypoint", então vamos para a prática.
</br></br>
<h2>Informações importantes</h2>
Antes precisamos entender rapidamente o que é um entrypoint, que em um contexto Docker é um arquivo de script shell que é frequentemente usado como ponto de entrada para um contêiner Docker quando ele é iniciado. Esse script é executado assim que o contêiner é iniciado e pode ser usado para realizar tarefas específicas antes de iniciar o aplicativo principal dentro do contêiner. No nosso caso ao criar esse arquivo no diretório: $ .docker/entrypoint.sh adicioamos a ele a seguinte estrutura:

```bash 
#!/bin/bash

npm install

npm run start:dev
```
Já o nosso docker-compose será o responsável por fazer a chamada se execução de inicializaação no contexto do entrypoint. Não abordaremos o que é um docker-compose, porém podemos dizer que ele é como um gerenciador de containers que são criados a partir das nossas imagens. Em sua estrutura vamos ver que ele possui um comando que faz a chamada do entrypoint:

```bash 
version: '3'

services:
  my-app:
    build:
      context: .
    container_name: my-app-project
    entrypoint: sh ./.docker/entrypoint.sh ## a chamada para o arquivo entrypoint é feita por meio desse comando.
    tty: true
    ports:
      - 3000:3000
    volumes:
      - .:/usr/my-app/app
```
Existem também uma outra abordagem utilizada para chamar o entrypoint a partir do arquivo Dockerfile através do comando <code>ENTRYPOINT: path</code>, porém aqui em nosso caso estou utilizando uma abordagem em que essa chamada é feita através de um compose.
</br></br>
<h2>Mantendo nosso gerenciador de pacotes sempre atualizado</h2>
Um ponto importante é que vamos aqui tratar da criação de um script que envolve lógica e a criação de um passo a passo, o que devemos ter em mente é que para criar scripts em alguns casos estamos falando da automatização de atividades as vezes do nosso dia a dia. Por exemplo, ao inicializarmos uma aplicação e executar um servidor com Node podemos utilizar simplesmente o comando <code>node filepath</code>, ao executar um script do próprio NPM como <code>npm run build</code>, <code>npm install | npm uninstall</code> estamos executando scripts que rodam dentro do package-manager do Node por debaixo dos panos, algo semelhante ao que criamos quando instalamos o Nodemon por exemplo que vai nos dar recursos como hotreload além de evitar execuções manuais, para isso criamos um "script { "start:dev": "nodemon filepath", "start": "node filepath" }" dentro de nosso package.json.
</br></br>
Entendido isso executamos nosso compose com o comando <code>docker-compose up -d && docker-compose logs -f <container_name></code>, que vai subir o container com nossa aplicação e em seguida executar os logs

```bash
my-app | up to date, audited 107 packages in 595ms
my-app |
my-app | 13 packages are looking for funding
my-app |   run `npm fund` for details
my-app |
my-app | found 0 vulnerabilities
my-app | npm notice
my-app | npm notice New major version of npm available 9.8.1. -> 10.1.0
my-app | npm noitice Changelog: https://github.com/npm/cli/releases/tag/v10.1.0
my-app | npm notice Run npm install -g npm@10.1.0 to update!
my-app |
my-app | > my-app@1.0.0. start:dev
my-app | > nodemon server.js
my-app |
my-app | [nodemon] 3.0.1
my-app | [nodemon] to restart at any time, enter `rs`
my-app | [nodemon] watching path(s): *.*
my-app | [nodemon] watching extensions: js,msj,cjs,json
my-app | [nodemon] starting `node server.js`
my-app | Server running on http://localhost:3000
```
Note que o ao subir o container o log nos mostra que a aplicação subiu normalmente, inclusive executando o Nodemon e trazendo a mensagem que aplicamos ao console.log no server de nossa aplicação, porém ela também nos mostra uma série de outras informações como quantidade de pacotes atualizados, vulnerabilidades que esses pacotes possam trazer para a nossa aplicação e a principal delas que é o "npm notice New major version of npm available 9.8.1. -> 10.1.0" essa informação é a que nos interessa. Isso em si é apenas um warning, mas notamos que em seguida o npm nos entrega a maneira de como "solucionar" isso que é atraves do comando "npm install -g npm@10.1.0", esse comando realiza o update de nossa versão atual para a versão disponibilizada.
</br></br>
Como vimos anteriormente a criação de scripts engloba a automatização de nossas necessidades do dia a dia, nesse caso ela diz respeito a atualização do npm em nossa máquina para que possamos manter nosso gerenciador de pacotes sempre atualizado evitando percalços na instalação de bibliotecas mais modernas e na gestão das existentes.
</br></br>
Como também sabemos é possível consultar uma versão com o comando <code>npm -v | npm --version</code> ou verificar se existe alguma nova versão disponível com o comando <code>npm show npm version</code>, e o que esses comandos fazem respectivamente são mostrar a versão atual Ex: v9.8.1 e verificar se existe uma nova versão Ex: v10.1.0. Sabendo disso temos a opção de manter o gerenciador de pacotes atualizado por conta própria utilizando os comandos <code>npm install -g npm@version</code>, <code>npm install -g npm@*</code> ou <code>npm install -g npm@latest</code> que respectivamente instalam globalmente uma versão especifica, todas as versão e pela current version, ou seja a versão mais atual. Tendo isso em mente partimos para a criação do nosso script.
</br></br>
Como verificamos no tópico anterior temos a opção de utilizar um entrypoint, que no contexto do Docker é o arquivo de entrada que vai executar ações pre-configuradas em sequencia de script uma vez que a imagem é criada, e é atraves dele que iremos realizar nossa ação de update do NPM. Lembre-se, é importantissimo desenvolver um senso analítico inclusive em cima das ações do seu dia a dia, programar em si é estar diante de uma sequencia repetitiva e conhecer essa sequencia é fundamental para te fazer avançar.
</br></br>
A situação é a seguinte:
<ul>
  <li>Ao criar uma imagem personalizada no Docker, tenho como base uma imagem já existente do Node</li>
  <li>Ao criar essa imagem necessito de um entrypoint</li>
  <li>A responsábilidade do entrypoint é executar os comandos de intalação do NPM (gerando uma node_modules) "npm install" e executar meu servidor de desenvolvimento "npm run start:dev"</li>
  <li>Preciso que ele sempre verifique a existencia de novas versões do NPM</li>
  <li>Se houver ele precisa atualizar, caso contrário segue o fluxo normal</li>
</ul>
Acabamos de criar um algoritmo que fará essa ação e agora precisamos implementá-lo. Adianto que será necessário entender o básico de terminal shell e alguma familiaridade com o Linux</br>
Caso queira é só clicar no link <a href="https://www.alura.com.br/conteudo/shellscripting">e estudar esse tópico</a></br>
Então vamos para o nosso entrypoint.sh realizar nossas configurações:</br></br>

<span>Estrutura atual do arquivo</span>
```bash
#!/bin/bash

# instala o npm gerando uma node_modules com os pacotes ja instalados,
# é importante realizar previamente a configuração de volumes no Docker.
npm install

# após instalar o NPM irá inicializar o script que executa a aplicação,
# nesse caso definido em nosso "package.json"
npm run start:dev
```
<span>Estrutura do arquivo com a configuração</span>
```bash
#!/bin/bash

# crie variáveis que irão armazenar as suas necessidades
# verificar a versão atual do NPM
current_npm_version=$(npm -v)

# verificar a versão do npm mais recente disponível
latest_npm_version=$(npm show npm version)

# aplique uma condicional que verifica as duas variáveis
if [ "$current_npm_version" != "$latest_npm_version" ];
then
  echo "Nova versão disponível $latest_npm_version, atualizando..."
  npm install -g npm@latest
  npm install
else
  npm install
fi

npm run start:dev
```
<h2>Observações</h2>
Utilizada a abordagem a cima aprendemos como fazer a atualização de nosso NPM dentro do container Docker, porém existem pontos que devemos levar em consideração:
<ul>
  <li>Mantenha também seu gerenciador de pacotes atualizado em seu ambiente local independente de container</li>
  <li>Se você estiver utilizando USER não-root em seu Dockerfile (o que é uma boa prática por motivos de segurança) você pode ter seus acessos de superusuario limitados, nesse caso recomenda-se que o update seja feito externamente ou com um usuario que permita esse tipo de operação.</li>
  <li>Sim é possível realizar esse update utilizando os poderes do USER não-root, porém é necessário realizar a sua inclusão no Cgroups do Docker pelo Dockerfile, em outras palavras seria como usar uma basuca para matar uma formiga, não vale muito a pena!</li>
</ul>
Concluídos estes pontos vemos que o Docker nos permiti acesso a todo tipo de ambiente dentro de diversas aplicações e sistemas operacionais em diversos recursos, Isso faz do Docker uma ferramenta barata e largamente utilizada para ambientes e infraestrutura. Por hoje aprendemos como utilizar uma parte desse poder que o Docker nos entrega para manter nossa aplicação atualizada dentro de seu próprio contexto. 
  
