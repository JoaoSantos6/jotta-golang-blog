---
title: "Meu Primeiro Nil Pointer Dereference em Go"
date: 2026-06-25
tags: ["erros-e-aprendizados", "fundamentos"]
draft: false
summary: "Registrando o primeiro panic real que encontrei: um nil pointer dereference causado por uma interface que parecia inicializada, mas não estava."
description: "Registrando o primeiro panic real que encontrei: um nil pointer dereference causado por uma interface que parecia inicializada, mas não estava."
---

Depois de uma semana estudando a sintaxe básica, tentei escrever meu primeiro
mini-projeto: um parser de configuração simples. Levou uns 20 minutos até eu
esbarrar num `panic: runtime error: invalid memory address or nil pointer
dereference`.

## O bug

```go
type Config struct {
	Nome *string
}

func main() {
	c := Config{}
	fmt.Println(*c.Nome) // panic aqui
}
```

`c.Nome` é um ponteiro (`*string`) que nunca foi inicializado — seu valor zero
é `nil`. Tentar desreferenciar (`*c.Nome`) um ponteiro `nil` é exatamente o
que causa o panic.

## A lição

Em Go, todo tipo tem um "valor zero" bem definido, e para ponteiros esse valor
é `nil`. Diferente de outras linguagens onde um "objeto vazio" pode ter um
estado utilizável, um ponteiro `nil` em Go não aponta para lugar nenhum — usar
sem checar antes é a receita para esse panic.

## Como resolvi

```go
func main() {
	c := Config{}
	if c.Nome != nil {
		fmt.Println(*c.Nome)
	} else {
		fmt.Println("nome não definido")
	}
}
```

Ou, melhor ainda para esse caso: usar `string` em vez de `*string`, já que o
valor zero de `string` é `""` — uma string vazia é um estado válido e seguro
de usar diretamente, sem checagem de nil.

Fica o aprendizado: só usar ponteiro quando eu realmente precisar distinguir
"valor não definido" de "valor zero" — na dúvida, o tipo por valor é mais
seguro.
