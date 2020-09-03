---
date: 2020-08-24
title: 'RESTful API em Golang - Parte 01'
slug: 'escrevendo_api_restful_golang_pt1'
timetoread: ''
tags: ['golang', 'api']
description: 'Vamos ver como a linguagem Golang é utilizada para construir APIs HTTP e como podemos estruturar nossa aplicação'
---

## Introdução

Go é uma linguagem de programção estática de código aberto e livre criada pela empresa Google que teve sua primeira entrega em 10 de novembro de 2009 e que está em sua versão 1.15 na data desta publicação. Se quiser saber mais sobre o que é o REST e sobre HTTP recomendo esse material da [**freecodecamp**](https://www.freecodecamp.org/news/restful-services-part-i-http-in-a-nutshell-aab3bfedd131/):link:.

Aqui gostaria de compartilhar com vocês sobre como eu estruturo API's HTTP Restful com Go, para sugestões de melhoria podem me mandar mensagem no twitter ou me comunicar por e-mail. :smile:

## Iniciando o projeto

Utilizaremos o gerenciador de dependencias [**Go Modules**](https://blog.golang.org/using-go-modules) que foi introduzido em 2019 como uma forma definitiva de gerenciamento de pacotes pela linguagem Go. Vamos utilizar o [**Gin**](https://github.com/gin-gonic/gin) como nosso framework HTTP, gosto do Gin por vir com HTTP logger por padrão.

Após criarmos o diretório do nosso projeto, inicializaremos o desenvolvimento com:

```sh
# go mod init github.com/{seu_usuario_no_github}/{nome_do_seu_projeto}

mkdir golang-rest-api && cd golang-rest-api
go mod init github.com/eclesiomelojunior/golang-rest-api
```

> Não existe uma forma correta para o nome que é informado como paramêtro para o **go mod init**, sigo esse padrão por estar mais acostumado com ele porém é permitido utilizar somente o nome do seu projeto se assim preferir.

Após esse comando, a raiz do seu projeto deve conter um arquivo chamado _go.mod_ esse arquivo armazenará o namespace, no nosso caso _github.com/eclesiomelojunior/golang-rest-api_, a versão do Go e os pacotes que utilizaremos para o desenvolvimento do nosso projeto. :rocket:

## Iniciando o servidor

Com nosso gerenciador de dependencias configurado é possivel adicionar pacotes externos, e vamos fazer isso com o Gin! Abra o terminal no mesmo diretório do arquivo _go.mod_ e execute o comando:

```sh
go get -u github.com/gin-gonic/gin
```

O seu arquivo _go.mod_ estará semelhante a esse:

```go
module github.com/eclesiomelojunior/golang-rest-api

go 1.14

require (
	github.com/gin-gonic/gin v1.6.3
)
```

Com as dependências configuradas podemos começar com os diretórios do nosso projeto. Vamos começar criando o diretório (pacote) **config** e dentro do diretório vamos criar o arquivo **config.go**

> Em Go não nos preocupamos com os nomes dos arquivos que estão dentro de um diretório, o que importa é o nome do diretório em si o qual simboliza um pacote, então é sempre importante existir pacotes com nomes indicando seu proposito e evite nomear pacotes com _utils_ ou _helpers_.

A configuração desse projeto terá somente a porta onde nossa aplicação levantará, ex. _:8080_

```sh
mkdir config && touch config.go
```

```go
// ./config/config.go
package config

// Config define a struct que terá a porta onde nossa aplicação levantará
type Config struct {
	Port string
}

// New retorna nossa struct
func New(port string) *Config {
	return &Config{
		Port: port,
	}
}
```

Após nosso arquivo de configuração vamos criar o pacote **server**, ele será o responsável por inicializar o servidor HTTP, definir as rotas e para quais controladores (handlers) elas resolvem, grupos de rotas e middlewares. Crie uma pasta no diretório raiz da aplicação com o nome **server** e dentro dessa pasta crie o o arquivo **server.go**.

```sh
mkdir server && touch server.go
```

A linguagem Go permite a criação de interfaces, então criaremos a interface **Server** que terá a assinatura do método **Run**

```go
//./server/server.go
package server

// Server interface
type Server interface {
  Run() error
}
```

Básicamente a função da nossa interface **Server** é levantar nossa aplicação, então vamos partir para a implementação dessa interface com a utilização da lib [**Gin**](https://github.com/gin-gonic/gin). Dentro do pacote (diretório) **server** vamos criar o arquivo **gin.go**.

```go
//./server/gin.go
package server

import (
  "github.com/eclesiomelojunior/golang-rest-api/config"
  "github.com/gin-gonic/gin"
)

type ginserver struct {
  engine *gin.Engine
  config *config.Config
}
```

Nesse arquivo **./server/gin.go** tem uma _struct_ chamada **ginserver**, que é privada para o pacote, ou seja, o à essa struct às suas propriedades e seus métodos são restritos somente para os arquivos desse pacote.

Agora, para que o ginserver obedeça ao contrato estabelecido pela interface vamos implementar o método **Run()**

```go
// ./server/gin.go
...

func NewGinServer(c *config.Config) Server {
	r := gin.Default()
	return &ginserver{r, c}
}

// ./server/gin.go
func (s *ginserver) Run() error {
  return s.engine.Run()
}
```

Pronto! Adicionamos ao arquivo **gin.go** uma fução **NewGinServer**, cria uma instancia do Gin e retorna a implementação da interface Server, e adicionamos na struct o método **Run**. Não precisamos indicar explicitamente que uma struct implementa uma interface, o simples fato de uma strcut ter todos os metodos requeridos por uma interface já basta para "_implementar a interface_"

Para finalizar a primeira parte iremos criar o _entrypoint_ da aplicação, para isso vamos criar o diretório **cmd** e dentro o pacote **api** e dentro do pacote vamos ter o arquivo **api.go**

```sh
mkdir -p cmd/api && touch cmd/api/api.go
```

```go
// ./cmd/api/api.go
package main

import (
	"flag"
	"log"

	"github.com/eclesiomelojunior/golang-rest-api/config"
	"github.com/eclesiomelojunior/golang-rest-api/server"
)

var port string

func init() {
	flag.StringVar(&port, "port", ":5000", "setup the application server port, default :5000")
	flag.Parse()
}

func main() {
	c := config.New(port)
	srv := server.NewGinServer(c)

	if err := srv.Run(); err != nil {
		log.Printf("application stoped: %s \n", err.Error())
	}
}
```

Esse pacote main é necessário para que o comando **go run** funcione. A função init tem a responsabilidade de inicializar a variável _port_ com a porta que o nosso servidor executará, ele busca o valor de uma flag que vamos informar no comando **go run**.

A função main executará imediatamente após a função init e criará uma instância da nossa config e do nosso server. Para levantar tudo voce pode executar o comando dentro do diretório _./cmd/api_:

```sh
cd ./cmd/api && go run api.go --port :8080
```

O servidor deverá levantar e escutar requisições em **http://localhosts:8080**

Na próxima parte vamos criar as rotas e um middleware para resolver problemas com o CORS. 🚀🚀
