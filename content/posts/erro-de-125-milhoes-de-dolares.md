---
title: "Tipagem Nominal: O Erro de 125 Milhões de Dólares que o Go Te Ajuda a Evitar"
date: 2026-07-26
tags: ["fundamentos", "tipos"]
draft: false
summary: "A história da sonda Mars Climate Orbiter, perdida por causa de uma confusão de unidades, e como o sistema de tipos de Go transforma esse tipo de bug em erro de compilação."
description: "A história da sonda Mars Climate Orbiter, perdida por causa de uma confusão de unidades, e como o sistema de tipos de Go transforma esse tipo de bug em erro de compilação."
---

## Uma sonda, dois times e uma unidade de medida

Em dezembro de 1998, a NASA lançou a *Mars Climate Orbiter*, uma sonda de 125
milhões de dólares, numa viagem de nove meses até Marte. Chegando lá, em
setembro de 1999, ela simplesmente sumiu.

Não foi explosão, não foi falha de motor, não foi meteoro. A sonda
funcionava perfeitamente. O problema estava no software — e era o tipo de
bug que qualquer um de nós já cometeu.

Um time (a Lockheed Martin) escreveu um código que devolvia a força dos
propulsores em **libras-força por segundo**, uma unidade do sistema imperial
americano. O outro time (a equipe de navegação da NASA) tinha um código que
lia esse número esperando **newtons por segundo**, do sistema métrico. Os
dois lados trocavam apenas o número, sem nunca dizer qual era a unidade.

Uma libra-força vale cerca de 4,45 newtons. Ou seja: cada valor que passava
de um sistema para o outro estava errado por um fator de 4,45 — mas ninguém
percebeu, porque na tela era só um número que parecia razoável. Depois de
nove meses acumulando pequenos desvios de trajetória, a sonda entrou na
atmosfera de Marte na altitude errada e se desintegrou.

Guarde essa frase, porque é o coração deste post:

> O número estava certo. O que faltou foi dizer **o que aquele número
> significava**.

---

## O que isso tem a ver com Go?

Tudo. Porque uma das primeiras coisas legais que você descobre estudando Go
é que a linguagem tem um jeito simples de resolver exatamente esse tipo de
confusão. Vamos por partes.

### Primeiro: o que é "tipo"?

Todo valor no seu programa tem um **tipo** — uma etiqueta que diz "isto é um
número inteiro", "isto é um texto", "isto é um valor verdadeiro/falso". Em
Go, os tipos básicos incluem:

- `int` — número inteiro (ex: `42`)
- `float64` — número com casa decimal (ex: `3.14`)
- `string` — texto (ex: `"olá"`)
- `bool` — verdadeiro ou falso (`true` / `false`)

Até aqui, nada de diferente de outras linguagens. A mágica vem agora.

### Segundo: em Go, você pode criar seus próprios tipos

Com uma única palavra, `type`, você inventa um tipo novo baseado em um que
já existe:

```go
type Celsius float64
type Fahrenheit float64
```

Lendo em voz alta: "*Celsius* é um novo tipo, feito em cima de `float64`" —
e a mesma coisa para *Fahrenheit*.

Por baixo dos panos, os dois **são** `float64`: guardam um número com casa
decimal, do mesmo jeitinho. Mas para o compilador do Go, eles agora são
**coisas diferentes**, com nomes diferentes. E é aí que mora a proteção.

### Terceiro: Go se recusa a misturar os dois

Veja o que acontece se você tentar tratar uma temperatura em Celsius como
se fosse Fahrenheit:

```go
var temp Celsius = 100
var f Fahrenheit = temp // ERRO: não compila!
```

O programa nem chega a rodar. O compilador te barra na hora com uma
mensagem parecida com:

```
cannot use temp (variable of type Celsius) as Fahrenheit value in variable declaration
```

Pare um segundo e repare no que aconteceu. Os dois valores são,
tecnicamente, `float64`. Poderiam se somar, se comparar, se misturar sem
reclamação nenhuma. Mas porque você deu **nomes** a eles, o Go entende que
são grandezas diferentes e **se recusa a deixar você confundi-las**.

É literalmente o bug da NASA sendo pego antes de existir. Se o software da
sonda fosse escrito assim, com um tipo `LibraForca` e um tipo `Newton`, a
tentativa de passar um no lugar do outro nem teria compilado. O erro de 125
milhões de dólares viraria uma linha vermelha no editor de código do
programador, resolvida em dois minutos.

---

## Um paralelo pra fixar

Imagine duas garrafas idênticas em cima da bancada. Uma tem água, a outra
tem álcool. No olho, são a mesma coisa: mesmo formato, mesmo líquido
transparente.

Se elas não têm rótulo, uma hora alguém vai beber da errada.

Criar um tipo em Go é **colar o rótulo na garrafa**. O líquido continua
parecido (os dois são `float64`), mas agora existe alguém — o compilador —
conferindo o rótulo toda vez e te impedindo de trocar uma pela outra por
engano.

---

## Isso não é teoria: o próprio Go usa esse truque

Você vai esbarrar nesse padrão logo nos primeiros programas. O exemplo mais
comum é o tipo `time.Duration`, que a biblioteca padrão usa para
representar intervalos de tempo. Por baixo, ele é só um número inteiro —
mas com nome próprio. Por isso você escreve assim:

```go
time.Sleep(5 * time.Second) // dorme por 5 segundos, bem claro
```

E **não** assim:

```go
time.Sleep(5) // isso dorme por 5 NANOSSEGUNDOS!
```

A unidade está embutida no tipo. O `5 * time.Second` carrega o significado
"segundos" junto com o número, exatamente o que faltou na *Mars Climate
Orbiter*. O código explica a si mesmo, e a chance de você errar a unidade
despenca.

---

## Bônus: seu tipo pode ganhar comportamento

Como em Go você criou um tipo de verdade (e não só um apelido), pode até
"ensinar" ele a fazer coisas. Por exemplo, ensinar `Celsius` a se converter
para `Fahrenheit`:

```go
func (c Celsius) ParaFahrenheit() Fahrenheit {
    return Fahrenheit(c*9/5 + 32)
}
```

Agora a conversão mora dentro do próprio tipo, no lugar certo:

```go
temp := Celsius(100)
fmt.Println(temp.ParaFahrenheit()) // 212
```

Repare que a conversão só acontece de propósito, com aquele
`Fahrenheit(...)` explícito. Você **pode** converter quando quiser — só não
vai fazer isso sem perceber. E essa diferença entre "por acidente" e "de
propósito" é tudo o que separa uma sonda em órbita de uma sonda virando
poeira em Marte.

---

## Resumo pra levar pra casa

- Em Go, `type NomeNovo TipoAntigo` cria um tipo **novo e distinto**, mesmo
  que por baixo os dois sejam iguais.
- O compilador **não deixa** você misturar tipos diferentes por engano — o
  erro aparece antes de o programa rodar.
- Isso transforma bugs de significado (como confundir unidades) em erros de
  compilação bobos e fáceis de corrigir.
- A própria biblioteca padrão usa isso o tempo todo, como em
  `time.Duration`.
- Bônus: seus tipos podem ter comportamento próprio, deixando cada regra no
  lugar certo.

A filosofia por trás de tudo isso, e que vale carregar no resto dos seus
estudos de Go: **o compilador não é seu inimigo, é seu revisor**. Cada
"chatice" dele é um problema a menos esperando por você em produção.

A NASA aprendeu do jeito caro. Com Go, você aprende de graça.
