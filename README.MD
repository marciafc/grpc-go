# Aplicação server gRPC em Go

- Projeto desenvolvido durante o curso FullCycle 3.0 para estudar sobre gRPC do lado do servidor

- Foi utilizado o Evans como client

- Banco de dados sqlite

- Go version 1.19.3

- Compilador do protocol buffer 3.6.1

## Preparação do ambiente

1. Instalar Go -> [Download and install](https://go.dev/doc/install)

2. Instalar o compilador do protocol buffer -> [Protocol Buffer Compiler Installation](https://grpc.io/docs/protoc-installation/) - Ensure compiler version is 3+

3. Instalar plugins Go para o compilador

```
// Este será utilizado para gerar as entidades / mensagens
$ go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28

// Este será utilizado para gerar as interfaces de comunicação usando gRPC
$ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2
```

4. No Visual Studio Code, instalar extensão **vscode-proto3** - (zxh404.vscode-proto3)

5. Instalar client [Evans](https://github.com/ktr0731/evans)

```
$ go install github.com/ktr0731/evans@latest  
```

## Setup do projeto

  - Inicializando
  
  ```  
  $ go mod init github.com/marciafc/grpc-go
  $ go mod tidy
  ``` 

  - Geração de código na pasta 'pb'
  
  ```    
  $ protoc --go_out=. --go-grpc_out=. proto/course_category.proto
  ```  

  - Criar tabela no sqlite
  
  ```  
  $ sqlite3 db.sqlite
  sqlite> create table categories (id string, name string, description string);
 
  // Validando se a tabela foi criada 
  sqlite> .tables
  categories  
  ```
  
  - Em outro terminal, baixar os pacotes
  
  ```  
  $ go mod tidy  
  ```  

## Executando a aplicação

```
// Rodar app Go
$ go run cmd/grpcServer/main.go
 
// Em outro terminal, executar o client Evans 
$ evans -r repl 
```

## Chamando os serviços de Category

- No terminal do Evans, selecionar o service

```
pb.CategoryService@127.0.0.1:50051> service CategoryService
```

### Criando uma category

```
// No terminal do Evans, chamar a criação de categoria
pb.CategoryService@127.0.0.1:50051> call CreateCategory

// Informar o name e a category
name (TYPE_STRING) => Categoria1
description (TYPE_STRING) => Descrição1

// O retorno será similar a este formato
{
  "description": "Descrição1",
  "id": "75182f19-4bd3-48b9-8b55-06104d865578",
  "name": "Categoria1"
}
```
 
### Listar todas as categorias

```
// No terminal do Evans, chamar a listagem de categorias
pb.CategoryService@127.0.0.1:50051> call ListCategories

// O retorno será similar a este formato
{
  "categories": [
    {
      "description": "Descrição1",
      "id": "75182f19-4bd3-48b9-8b55-06104d865578",
      "name": "Categoria1"
    }
  ]
}
```

### Buscar category por id

```
// No terminal do Evans, chamar a busca de categoria por id
pb.CategoryService@127.0.0.1:50051> call GetCategory

// Informar o id
id (TYPE_STRING) => 75182f19-4bd3-48b9-8b55-06104d865578

// O retorno será similar a este formato
{
  "description": "Descrição1",
  "id": "75182f19-4bd3-48b9-8b55-06104d865578",
  "name": "Categoria1"
}
```

### Criar várias categories - usando stream

  - Irá receber várias requisições para salvar e irá retornar tudo ao término

```
// No terminal do Evans, chamar a criação de categorias por stream
pb.CategoryService@127.0.0.1:50051> call CreateCategoryStream

// Informar o name e a description da primeira categoria
name (TYPE_STRING) => Categoria 1 stream
description (TYPE_STRING) => Desc 1

// Informar o name e a description da segunda categoria
name (TYPE_STRING) => Categoria 2 stream
description (TYPE_STRING) => Desc 2

// Informar o name e a description da terceira categoria
name (TYPE_STRING) => Categoria 3 stream
description (TYPE_STRING) => Desc 3

// Para parar de enviar: control + D
name (TYPE_STRING) => 

// O retorno será similar a este formato - com todas as categorias de uma só vez
{
  "categories": [
    {
      "description": "Desc 1",
      "id": "126e06a8-4dd6-4e8c-8494-6ab58d6d77f8",
      "name": "Categoria 1 stream"
    },
    {
      "description": "Desc 2",
      "id": "91709589-76ff-494b-bfed-7fd80b2616cd",
      "name": "Categoria 2 stream"
    },
    {
      "description": "Desc 3",
      "id": "0219754b-ec63-4984-858d-99d71368b820",
      "name": "Categoria 3 stream"
    }
  ]
}
```

### Criar várias categories - usando stream bidirecionais

  - Irá receber as requisições para salvar e irá retornar logo em seguida (a cada requisição)

```
// No terminal do Evans, chamar a criação de categorias por stream bidirecional
pb.CategoryService@127.0.0.1:50051> call CreateCategoryStreamBidirectional

// Informar o name e a description da primeira categoria
name (TYPE_STRING) => Cat 1 bidirecional
description (TYPE_STRING) => Desc 1

// O retorno da primeira categoria será similar a este formato
name (TYPE_STRING) => {
  "description": "Desc 1",
  "id": "96d070ca-07d2-473c-b492-f9304aedff0e",
  "name": "Cat 1 bidirecional"
}

// Informar o name e a description da segunda categoria
name (TYPE_STRING) => Cat 2 bidirecional
description (TYPE_STRING) => Desc 2

// O retorno da segunda categoria será similar a este formato
name (TYPE_STRING) => {
  "description": "Desc 2",
  "id": "66231b2f-2371-4a28-8493-6eaa15b109ed",
  "name": "Cat 2 bidirecional"
}

// Informar o name e a description da terceira categoria
name (TYPE_STRING) => Cat 3 bidirecional
description (TYPE_STRING) => Desc 3

// O retorno da terceira categoria será similar a este formato
name (TYPE_STRING) => {
  "description": "Desc 3",
  "id": "c6541352-e9fd-4a3b-92d1-5716d7da7f52",
  "name": "Cat 3 bidirecional"
}

// Informar control D para parar de enviar
name (TYPE_STRING) =>
```