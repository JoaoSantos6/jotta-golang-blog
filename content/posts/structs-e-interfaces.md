---
title: "Structs e Interfaces: Como Go Faz Composição"
date: 2026-07-02
tags: ["fundamentos"]
draft: false
summary: "Anotações da primeira semana estudando os tipos compostos de Go — structs, embedding e como interfaces são satisfeitas implicitamente."
---

Vindo de linguagens com herança clássica, o modelo de composição de Go foi a
primeira coisa que quebrou minhas expectativas — de um jeito bom.

## Struct simples

```go
type Autor struct {
	Nome  string
	Email string
}

func (a Autor) Apresentacao() string {
	return fmt.Sprintf("%s <%s>", a.Nome, a.Email)
}
```

## Embedding em vez de herança

Go não tem herança de classes. Em vez disso, você "embeda" um struct dentro de
outro, e os campos/métodos do struct embedado ficam disponíveis diretamente:

```go
type Post struct {
	Autor
	Titulo string
}

p := Post{
	Autor:  Autor{Nome: "João", Email: "joao@example.com"},
	Titulo: "Primeiro post",
}

fmt.Println(p.Apresentacao()) // método "herdado" via embedding
```

## Interfaces satisfeitas implicitamente

O que mais me surpreendeu: não existe `implements` em Go. Se um tipo tem os
métodos que uma interface exige, ele já satisfaz a interface — sem declaração
explícita:

```go
type Apresentavel interface {
	Apresentacao() string
}

func Imprimir(a Apresentavel) {
	fmt.Println(a.Apresentacao())
}

// Autor satisfaz Apresentavel automaticamente, sem "implements"
Imprimir(Autor{Nome: "João", Email: "joao@example.com"})
```

Isso muda a forma de desenhar código: em vez de planejar hierarquias, você
define comportamentos pequenos (interfaces enxutas) e deixa qualquer tipo que
já tenha os métodos certos se encaixar.
