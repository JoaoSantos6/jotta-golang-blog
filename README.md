# Blog de Estudos em Go

Blog pessoal para documentar meu processo de aprendizado da linguagem Go: anotações, exercícios, pequenos projetos e os erros (e soluções) encontrados pelo caminho.

A ideia é simples: escrever sobre o que estou aprendendo enquanto ainda estou aprendendo — tanto para consolidar o conhecimento quanto para servir de referência para quem está começando também.

🔗 [JoaoSantos6.github.io/jotta-golang-blog](https://JoaoSantos6.github.io/jotta-golang-blog/)

## Stack

- [Hugo](https://gohugo.io/) — gerador de site estático
- [PaperMod](https://github.com/adityatelange/hugo-PaperMod) — tema
- Markdown para o conteúdo dos posts
- GitHub Actions — build e deploy automáticos
- GitHub Pages — hospedagem

## Rodando localmente

```bash
git clone --recurse-submodules https://github.com/JoaoSantos6/jotta-golang-blog.git
cd jotta-golang-blog
hugo server -D
```

O site fica disponível em `http://localhost:1313`.

## Criando um novo post

```bash
hugo new content/posts/nome-do-post.md
```

## Deploy

O deploy é automático: todo push na branch `main` dispara um workflow do GitHub Actions que builda o site com Hugo e publica no GitHub Pages.
