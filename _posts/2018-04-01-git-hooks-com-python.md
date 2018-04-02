---
layout: post
title: Git Hooks com Python
subtitle: "print(“Hello World of Devs!”)"   
tags: [git, hooks, python]
---

#### print(“Hello World of Devs!”)

Depois de um tempo sem escrever, e uma migração para um blog pessoal saindo do [Medium](https://medium.com/@lucas_souto), resolvi voltar escrever sobre as coisas que venho estudando para compartilhar um pouco do conhecimento que vou adiquirindo no dia a dia e que talvez seja útil para mais pessoas. 

Como de costume para acompanhar a leitura trago para vocês uma banda que conheci recentemente e estou ouvindo enquanto escrevo esse post, [Seu Pereira e Coletivo 401](https://open.spotify.com/artist/5dk0R5am4zMYmG6uyZwfkW?si=gCCOGyIsQWOmNcaQzH_zuA) é uma banda paraibana com estilos musicais que giram em torno do rock e MPB atual, não irão se arrepender de ouvir.

## Hooks do Git

Recentemente fui apresentado aos [hooks do git](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) enquanto tentava melhorar meus códigos, deixá-los mais limpos e legíveis e usar boas práticas para os projetos que venho tentando tocar para frente. Os hooks são scripts executados em algum momento da sua interação com o seu repositório git, tais como commit, push, pull, merge, rebase, etc. Essa possibilidade de você poder ter um workflow configurável no git te proporciona a flexibilidade de usar padrões para os seus projetos, por exemplo no `prepare-commit-msg` utilizar um  modelo de mensagem padrão para determinados casos, como em um merge. Vai da necessidade ou criatividade do desenvolvedor.

Para que se possa utilizar os hooks é bom primeiro conhecermos um pouco o diretório `.git`, ele é criado quando inicializamos nosso versionamento com `git init`, nele tem todos os arquivos que o git precisa para trabalhar, a pasta `hooks` contém alguns arquivos com terminação .sample, para que eles funcionem deve ser retirado o `.sample` do final do nome do script, os nomes desses arquivos já dizem por si só o seu papel no nosso workflow.

```
➜  .git git:(master) tree -L 1 hooks 
hooks
├── applypatch-msg.sample
├── commit-msg.sample
├── fsmonitor-watchman.sample
├── post-update.sample
├── pre-applypatch.sample
├── pre-commit.sample
├── prepare-commit-msg.sample
├── pre-push.sample
├── pre-rebase.sample
├── pre-receive.sample
└── update.sample

0 directories, 11 files
```

São diversas as opções que podem ser usadas, o `pre-commit` por exemplo é executado antes mesmo da mensagem do commit ser salva e o `post-commit` (que também é um script válido mas não está na pasta que vem padrão) que irá rodar depois que o commit é executado, muito legal para alguma mensagem personalizada depois de commitar algo.
Abrindo um deles notamos que são escritos em Shell Script.

```
#!/bin/sh
#layout: page
title: "About" 
subtitle: "This is a subtitle"   
feature-img: "assets/img/sample.png" 
permalink: /about.html               # Set a permalink your your page
hide: true                           # Prevent the page title to appear in the navbar
tags: [sample, markdown, html]

# An example hook script to prepare a packed repository for use over
# dumb transports.
#
# To enable this hook, rename this file to "post-update".

exec git update-server-info
```

## Python e Hooks

Mas não sabemos para onde vai essa sintaxe (pelo menos eu não sei rsrs) e ai que entra nosso querido Python, podemos escrever esses scripts em Python e irão rodar perfeitamente da mesma maneira que rodariam em Shell Script, além dessas duas linguagens também é possível utilizar outras linguagens.

Um dos hooks que mais utilizo é o `pre-push`, como uso bastante linters para analizar meus códigos e sempre que eu terminava o que tinha feito eu rodava os linters isso acabou se tornando chato e quis automatizar, até mesmo para não esquecer de rodar os linters e subir um "código sujo" para produção.

Para checar meus códigos utilizo o [Pylama](https://github.com/klen/pylama), ele é um wrapper de ferramentas que ajudam na hora de analisar o código. Executando `pylama` no terminal ele irá executar os linters no seu diretório.

Um hook escrito em python tem mais ou menos essa aparência:

{% highlight python %}
#!/usr/bin/env python
import os
import sys

if os.system('pylama') != 0:
    print('Não Passou')
    sys.exit(1)
else:
    print('Passou')
{% endhighlight %}

Bem simples, não? Para quem já conhece python não é problema algum entender esse código, mas vamos dar uma explicadinha. Na primeira linha é indicado qual a linguagem que aquele script será executado, depois pode ser escrito toda a sua "estratégia" para seu hook, no caso do exemplo acima eu só analiso se não ouve nenhum erro ao executar o comando `pylama` no terminal, se retornar algum valor diferente de zero é porque aconteceu algo inesperado e ele termina a execução.

Se utilizarmos esse código no `pre-commit` ele irá executar os linters e se passar vai finalizar o commit, se der algum erro ele aborta a operação antes de commitar.

```
(teste-git-LhWtE8kx) ➜  teste-git git:(master) ✗ git add teste.py 
(teste-git-LhWtE8kx) ➜  teste-git git:(master) ✗ git commit -m "Test pre-commit hook" 
teste.py:5:1: W391 blank line at end of file [pycodestyle]
Não Passou
```

Quando tentamos commitar nosso `teste.py` ele viu que o arquivo não condizia com o codestyle e abortou a operação, não commitando aquele código e printando a mensagem `Não Passou` que botamos no hook, se consertarmos esse style ele irá passar.

```
(teste-git-LhWtE8kx) ➜  teste-git git:(master) ✗ vim teste.py                    
(teste-git-LhWtE8kx) ➜  teste-git git:(master) ✗ git commit -am "Test pre-commit hook"
Passou
[master ca72ff0] Test pre-commit hook
 1 file changed, 1 insertion(+), 3 deletions(-)
```

Isso foi um exemplo muito simplório do que podemos fazer com hooks e python, pode ter coisas bem mais complexas ai, testem e comecem usar para melhorar algumas rotinas do seu desenvolvimento que irá ajudar bastante.

Quaisquer perguntas ou sugestões fiquem à vontade para fazerem nos comentários.