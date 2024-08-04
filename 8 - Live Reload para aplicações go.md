# Como melhorar a sua produtividade em aplicações GO
</hr>

Se você utiliza Go na construção de suas aplicações deve saber que a cada alteração em seu projeto utilizamos o comando <code>go run [path/file]</code> para rebuildar a nossa estrutura e ver os resultados aplicados, além também do comando <code>go build [path/file]</code>. Em resumo ambos os comandos tem papeis distintos:</br>

<code>go build [path/file]</code>: É utilizado para compilar o código-fonte em um projeto binário executável, ele verifica e compila o código, mas não o executa automaticamente, para isso precisamos executá-lo separadamente.</br>
<code>go run [path/file]</code>: Compila o código-fonte e o executa, porém não gera um projeto binário executável de forma permanente.</br>

Falando no quesito produtividade, utilizar o comando go run não é algo muito humanamente performático, pois a cada alteração é preciso parar o servidor go e chamar "go run" para refletir as suas alterações, logo em desenvolvimento isso pode se tornar algo cansativo. Porém não é o fim! Existe uma dependencia que nos possibilita utilizar o recurso de live reload e tornar o trabalho bem menos cansativo e que trás maior produtividade em seu desenvolvimento.</br>

<strong>Para isso vamos utilizar o AIR</strong></br>
Bem sabemos que a linguagem Go possui uma comunidade bem empenhada e atuante e com pessoas que se preocupam construir soluções eficientes para melhorar o dia a dia do desenvolvedor Go. Para isso utilizamos a ferramenta <a href="https://github.com/air-verse/air">air</a> que foi desenvolvida (segundo descrito em seu repositório) com a motivação de criar um live-reload mais performatico e fflexível se comparado aos fornecidos pelo framework gin-gonic, por exemplo.</br></br>

<strong>Instalando o Air globalmente no $GOPATH</strong></br>
Por ser uma ferramente escrita em Go podemos utilizar o install para isso.
Algumas de suas principais features são:
* Logs de output coloridos
* Customização de build ou qualquer commando
* Suporte para excluding de subdiretórios
* Habilita o watch mode para a crição de novos diretórios depois que o Air é iniciado

```bash
## A partir da versão go1.22 ou maior
$ go install github.com/air-verse/air@latest

## Instalação via shell no binario diretamente para o GOPATH $(go env GOPATH)/bin/air
curl -sSfL https://raw.githubusercontent.com/air-verse/air/master/install.sh | sh -s -- -b $(go env GOPATH)/bin

# ou instalar direto no diretorio ./bin/
curl -sSfL https://raw.githubusercontent.com/air-verse/air/master/install.sh | sh -s

air -v
```
Para maiores informações sobre a ferramente Air consulte a <a href="https://github.com/air-verse/air">documentação no repositório</a></br>

<strong>Passos de uso do live reload</strong></br>
Uma vez instalado o Air a sua utilização é algo bem simples. Primeiro certifique-se de que sua instalação ocorreu corretamente.
```bash
## Para verificar a versão instalada
$ air -v
  __    _   ___
 / /\  | | | |_)
/_/--\ |_| |_| \_ v1.52.3, built with Go go1.22.4
```
Para executar o Air e utilizar o recurso de live-reload é bastante simples. Para isso temos que executar o Air
```bash
## Para verificar a versão instalada
$ air init
  __    _   ___
 / /\  | | | |_)
/_/--\ |_| |_| \_ v1.52.3, built with Go go1.22.4

.air.toml file created to the current directory with the default settings
```
Ao executar o comando acima será criado um arquivo chamado <code>.air.toml</code> no diretório atual onde o comando foi executado.
O arquivo .air.toml gerado pelo comando air init é um arquivo de configuração para a ferramenta Air em seu recurso de live reloading de aplicações Go durante o desenvolvimento. Este arquivo define parâmetros como os diretórios e arquivos que devem ser monitorados por mudanças, comandos de build e execução, variáveis de ambiente, e outras opções que controlam como a aplicação deve ser recompilada e reiniciada automaticamente quando você faz alterações no código.
<strong>Observações importantes</strong></br>
Uma vez instalado a Golang necessita da execução da extensão oficial GO no VS Code para processar alguns recursos e ferramentas que a linguagem
depende para funcionar completamente. Para isso após instalar a extensão utilize o comando CTRL + SHIFT + P e procure pela opção >Go: Install/Update Tools
selecione todas as ferramentas necessárias para o funcionamento da linguagem.
```bash
root = "."                                                           # Diretório raiz do projeto
testdata_dir = "testdata"                                            # Diretório onde os dados de teste são armazenados
tmp_dir = "tmp"                                                      # Diretório temporário onde os arquivos compilados serão armazenados
[build]
  args_bin = []                                                      # Argumentos adicionais para o comando de build (geralmente vazio)
  bin = "./tmp/main"                                                 # Caminho do binário gerado após o build
  cmd = "go build -o ./tmp/main ."                                   # Comando para compilar o projeto
  delay = 1000                                                       # Tempo de espera em milissegundos entre detecção de mudanças e o rebuild
  exclude_dir = ["assets", "tmp", "vendor", "testdata"]              # Diretórios que serão ignorados durante a detecção de mudanças
  exclude_file = []                                                  # Arquivos específicos que serão ignorados durante a detecção de mudanças
  exclude_regex = ["_test.go"]                                       # Expressões regulares para ignorar certos arquivos (por exemplo, arquivos de teste)
  exclude_unchanged = false                                          # Se `true`, não recompila quando não houver mudanças no código
  follow_symlink = false                                             # Seguir links simbólicos durante a detecção de mudanças
  full_bin = ""                                                      # Caminho completo para o binário a ser executado (geralmente vazio)
  include_dir = []                                                   # Diretórios específicos a serem incluídos na detecção de mudanças
  include_ext = ["go", "tpl", "tmpl", "html"]                        # Extensões de arquivos a serem monitoradas por mudanças
  include_file = []                                                  # Arquivos específicos a serem monitorados por mudanças
  kill_delay = "0s"                                                  # Tempo para esperar antes de forçar o encerramento do processo antigo ao reiniciar
  log = "build-errors.log"                                           # Nome do arquivo de log de erros de build
  poll = false                                                       # Usar polling em vez de monitoramento de sistema de arquivos nativo (desabilitado por padrão)
  poll_interval = 0                                                  # Intervalo de polling em milissegundos, se o polling estiver habilitado
  post_cmd = []                                                      # Comandos a serem executados após o comando de build
  pre_cmd = []                                                       # Comandos a serem executados antes do comando de build
  rerun = false                                                      # Recompilar e reiniciar o processo quando um sinal for recebido
  rerun_delay = 500                                                  # Tempo de espera antes de recompilar ao receber um sinal
  send_interrupt = false                                             # Enviar um sinal de interrupção (SIGINT) ao invés de SIGTERM
  stop_on_error = false                                              # Parar o processo em caso de erro de build
[color]
  app = ""                                                           # Cor para o aplicativo em execução
  build = "yellow"                                                   # Cor para mensagens de build
  main = "magenta"                                                   # Cor para mensagens principais
  runner = "green"                                                   # Cor para o runner (executor)
  watcher = "cyan"                                                   # Cor para o watcher (monitor de mudanças)
[log]
  main_only = false                                                  # Mostrar apenas logs principais
  time = false                                                       # Incluir timestamp nos logs
[misc]
  clean_on_exit = false                                              # Limpar arquivos temporários ao sair
[proxy]
  app_port = 0                                                       # Porta do aplicativo ao usar um proxy reverso
  enabled = false                                                    # Habilitar proxy reverso
  proxy_port = 0                                                     # Porta do proxy reverso
[screen]
  clear_on_rebuild = false                                           # Limpar a tela ao reconstruir
  keep_scroll = true                                                 # Manter o scrollback no terminal
```
</br>

Observe o watch mode em funcionamento. Conforme o Air identifica alterações e o comando de Salvar <code>CTRL + S</code> do VS Code é acionado logs são disparados apresentando o arquivo que sofreu alteração.
![image](https://github.com/user-attachments/assets/94222416-6eab-4b84-852b-d800e3e4b21e)
</br>

Com isso eliminamos a necessidade de derrubar o servidor e restarta-lo a todo momento conforme as implementações vão sendo realizadas no contexto de desenvolvimento de uma aplicação Go.
 
