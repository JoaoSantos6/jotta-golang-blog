---
title: "Entendendo Goroutines na Prática"
date: 2026-07-09
tags: ["concorrência", "goroutines"]
draft: false
summary: "O que aprendi estudando concorrência em Go essa semana: goroutines, o runtime scheduler e as primeiras armadilhas."
---

Comecei a semana tentando entender por que `go func() {...}()` parece mágica
demais. Não é: goroutines são funções que rodam de forma concorrente,
gerenciadas pelo próprio runtime do Go, não pelo sistema operacional
diretamente.

## O exemplo mais simples

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	go func() {
		fmt.Println("rodando em outra goroutine")
	}()

	time.Sleep(100 * time.Millisecond)
	fmt.Println("main terminando")
}
```

Sem o `time.Sleep`, a função `main` pode terminar antes da goroutine sequer
rodar — o programa simplesmente encerra. Foi minha primeira armadilha: achar
que `go` bloqueia até a goroutine terminar. Não bloqueia.

## `sync.WaitGroup` em vez de `time.Sleep`

`time.Sleep` é gambiarra para aprendizado, não solução. O jeito correto é usar
`sync.WaitGroup` para esperar a goroutine terminar de verdade:

```go
var wg sync.WaitGroup
wg.Add(1)

go func() {
	defer wg.Done()
	fmt.Println("rodando em outra goroutine")
}()

wg.Wait()
fmt.Println("main terminando")
```

## Próximos passos

Semana que vem quero entender `channels` e como eles substituem locks
explícitos na maioria dos casos — o "share memory by communicating" que tanto
aparece na documentação oficial.
