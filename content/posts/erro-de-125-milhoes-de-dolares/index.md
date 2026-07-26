---
title: "Tipagem Nominal: O Erro de 125 Milhões de Dólares que o Go Nos Ajuda a Evitar"
date: 2026-07-26
tags: ["fundamentos", "tipos"]
draft: false
image: "cover.jpg"
summary: "A história da sonda Mars Climate Orbiter, perdida por causa de uma confusão de unidades, e como o sistema de tipos de Go transforma esse tipo de bug em erro de compilação."
description: "A história da sonda Mars Climate Orbiter, perdida por causa de uma confusão de unidades, e como o sistema de tipos de Go transforma esse tipo de bug em erro de compilação."
---

## Uma sonda, dois times e uma unidade de medida

*Imagem: [O Erro de $ 125 Milhões – Missão Mars Climate Orbiter, GPET Física](https://share.google/AjyCgqYn2lhYF8h6Y)*

Setembro de 1999. Depois de nove meses de viagem, uma sonda de **125
milhões de dólares** chega em Marte, e some. Sem explosão. Sem meteoro.
Sem falha de motor. O software funcionava perfeitamente.

O culpado: um número que trocou de mãos entre dois times sem ninguém dizer
**em que unidade** ele estava. Um lado falava em libras-força. O outro
ouvia newtons. A diferença (um fator de 4,45) foi se acumulando em
silêncio até a sonda mergulhar na atmosfera errada.

> O número estava certo. O que faltou foi dizer **o que aquele número
> significava**.

---

## O que isso tem a ver com Go?

Tudo. Porque uma das primeiras coisas legais que descobrimos estudando Go
é que a linguagem tem um jeito simples de resolver exatamente esse tipo de
confusão. Vamos por partes.

### Primeiro: o que é "tipo"?

Todo valor no nosso programa tem um **tipo** (uma etiqueta que diz "isto é um
número inteiro", "isto é um texto", "isto é um valor verdadeiro/falso"). Em
Go, os tipos básicos incluem:

- `int` (número inteiro, ex: `42`)
- `float64` (número com casa decimal, ex: `3.14`)
- `string` (texto, ex: `"olá"`)
- `bool` (verdadeiro ou falso, `true` / `false`)

Até aqui, nada de diferente de outras linguagens. A mágica vem agora.

### Segundo: em Go, podemos criar nossos próprios tipos

Com uma única palavra, `type`, inventamos um tipo novo baseado em um que
já existe:

```go
type Celsius float64
type Fahrenheit float64
```

Lendo em voz alta: "*Celsius* é um novo tipo, feito em cima de `float64`",
e a mesma coisa para *Fahrenheit*.

Por baixo dos panos, os dois **são** `float64`: guardam um número com casa
decimal, do mesmo jeitinho. Mas para o compilador do Go, eles agora são
**coisas diferentes**, com nomes diferentes. E é aí que mora a proteção.

### Terceiro: Go se recusa a misturar os dois

Vamos ver o que acontece se tentarmos tratar uma temperatura em Celsius
como se fosse Fahrenheit:

```go
var temp Celsius = 100
var f Fahrenheit = temp // ERRO: não compila!
```

O programa nem chega a rodar. O compilador nos barra na hora com uma
mensagem parecida com:

```
cannot use temp (variable of type Celsius) as Fahrenheit value in variable declaration
```

Vamos parar um segundo e reparar no que aconteceu. Os dois valores são,
tecnicamente, `float64`. Poderiam se somar, se comparar, se misturar sem
reclamação nenhuma. Mas porque demos **nomes** a eles, o Go entende que
são grandezas diferentes e **não deixa a gente confundir os dois**.

É literalmente o bug da NASA sendo pego antes de existir. Se o software da
sonda fosse escrito assim, com um tipo `LibraForca` e um tipo `Newton`, a
tentativa de passar um no lugar do outro nem teria compilado. O erro de 125
milhões de dólares viraria uma linha vermelha no editor de código do
programador, resolvida em dois minutos.

---

## Isso não é teoria: o próprio Go usa esse truque

Vamos esbarrar nesse padrão logo nos primeiros programas. O exemplo mais
comum é o tipo `time.Duration`, que a biblioteca padrão usa para
representar intervalos de tempo. Por baixo, ele é só um número inteiro (mas
com nome próprio). Por isso escrevemos assim:

```go
time.Sleep(5 * time.Second) // dorme por 5 segundos, bem claro
```

E **não** assim:

```go
time.Sleep(5) // isso dorme por 5 NANOSSEGUNDOS!
```

A unidade está embutida no tipo. O `5 * time.Second` carrega o significado
"segundos" junto com o número, exatamente o que faltou na *Mars Climate
Orbiter*. O código explica a si mesmo, e a chance de errarmos a unidade
despenca.

---

## Dito isso...

- Em Go, `type NomeNovo TipoAntigo` cria um tipo **novo e distinto**, mesmo
  que por baixo os dois sejam iguais.
- O compilador **não nos deixa** misturar tipos diferentes por engano (o
  erro aparece antes de o programa rodar).
- Isso transforma bugs de significado (como confundir unidades) em erros de
  compilação bobos e fáceis de corrigir.
- A própria biblioteca padrão usa isso o tempo todo, como em
  `time.Duration`.

A filosofia por trás de tudo isso, e que vale carregar no resto dos nossos
estudos de Go: **o compilador não é nosso inimigo, é nosso revisor**. Cada
"chatice" dele é um problema a menos esperando por nós em produção.

A NASA aprendeu do jeito caro. Com Go, aprendemos de graça.
