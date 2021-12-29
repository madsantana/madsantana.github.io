---
title: Testando a Segurança de Aplicações Ruby on Rails
author: Madsantana, aka, Marco Antonio Damaceno
date: 2021-12-29 13:24 +0300
categories: [Blogging]
tags: [tutoriais, tutorial, ruby, rails, segurança da informação, hacking, vulnerabilidade, pentest, hakiri, brakeman, devsecops]
pin: true
---
![Ruby Badge](https://img.shields.io/badge/-Ruby-red)
![Rails Badge](https://img.shields.io/badge/-Rails-green)

Salve galera! Como vocês estão?! Faz um bom tempo que não escrevo aqui, né?

<div style="text-align: justify">
Com tantas vulnerabilidades, algumas do tipo Zero-day, surgindo todos os dias, a tendência é que acabamos ficando apreensivos quanto as possíveis vulnerabilidades que possam existir em nossas aplicações Ruby on Rails.

Então invariavelmente surge a dúvida, como testar nossas aplicações RoR?

Rspec infelizmente não faz este tipo de trabalho, testando apenas a sintaxe. Então temos que procurar outras alternativas. Um pentest comum, usando as ferramentas mais conhecidas do mercado, como burpsuite, metasploit, dentre outros, também não ajudam muito, pois são ferramentas (por melhores que sejam) genéricas, não se aplicando para aplicações Rails.

Neste caso, após alguns bons momentos de pesquisa intensa, descobri duas gems bem interessantes, as quais acabei fazendo um fork no meu github, a fim de contribuir com estes projetos.

São elas:

a gem brakeman, e a gem hakiri.

As gems podem ser encontradas em seus sites oficiais abaixo:</div>

> Brakeman Scanner: <https://brakemanscanner.org/>

> Hakiri: <https://github.com/hakirisec/hakiri_toolbelt>

## Que tal praticarmos?!

<div style="text_align: justify">
A partir daqui, vamos criar uma aplicação básica usando o ruby 3 e o rails, para que possamos vasculhar a mesma em busca de possíveis vulnerabilidades. Num primeiro momento, vamos focar mais na gem brakeman, e veremos um pouco do uso da gem hakiri.

Resumindo, a gem brakeman fez uma varredura a nível de código fonte no projeto, retornando ao final um relatório berm extenso, que inclusive pode ser gerado em html, vale realmente a pena.

A gem hakiri faz um trabalho mais no "entorno" do projeto, procurando por possíveis vulnerabilidades no sistema operacional, nas versões do ruby que estão instaladas na máquina e em outras aplicações como apache, passenger, etc.</div>

|Ferramenta|Versão|Observações|
|:---|:--|---:|
|Ruby |3.0.1.64 | - |
|Rails | 6.1.4| - |
|Gem Beakeman | - | - |
|Gem Hakiri | - | - |


Vamos preparar o ambiente e criar o projeto, utilizando o Rails, para isso, execute os passos a seguir:

```Terminal
$rails new my_security_app
```
![Criando a Aplicação]({{ "/assets/img/tutoriais/security/security000.jpg" | relative_url }})

Vamos para a pasta que foi criada para o projeto:

```Terminal
$cd my_security_app
```

Vamos subir a aplicação uma primeira vez para nos certificar de que o Rails fez tudo certinho:

```Terminal
$rails s
```
![Desktop View]({{ "/assets/img/tutoriais/security/security001.jpg" | relative_url }})

Se tudo ocorreu bem até aqui, você deve vizualisar algo como a imagem abaixo:

![Desktop View]({{ "/assets/img/tutoriais/security/security002.jpg" | relative_url }})

Maravilha, nosso app está funcionando! Mas, será que ele está seguro? É o que vamos descobrir a seguir.

## Instalando a gem Brakeman

Não é necessário adicionar a gem no Gemfile, neste caso, estando na pasta do projeto, basta rodar o comando abaixo:

```Terminal
$gem install brakeman
```
![Desktop View]({{ "/assets/img/tutoriais/security/security003.jpg" | relative_url }})

A partir daqui não é necessário muita coisa, basta apenas, após instalar a gem, executá-la:

```Terminal
$brakeman
```
Ou, caso queira que o scanner gere um relatório html, rode-o da seguinte maneira:

```Terminal
$brakeman -o my_security_app_report.html
```

![Desktop View]({{ "/assets/img/tutoriais/security/security004.jpg" | relative_url }})

![Desktop View]({{ "/assets/img/tutoriais/security/security005.jpg" | relative_url }})


## Dando aquele check no relatório final

Conforme podemos ver nas próximas imagens, o brakeman gera um relatório bem completo e detalhado quando encontra alguma vulnerabilidade:

![Desktop View]({{ "/assets/img/tutoriais/security/security006.jpg" | relative_url }})

![Desktop View]({{ "/assets/img/tutoriais/security/security007.jpg" | relative_url }})

Aqui, rodei com a opção de criar um arquivo html, na imagem abaixo vemos para o nosso app criado para este tutorial, e notamos que aplicações criadas com ruby 3 rails 6 praticamente não apresentam  vulnerabilidades.

![Desktop View]({{ "/assets/img/tutoriais/security/security012.jpg" | relative_url }})

E na próxima imagem, rodei o brakeman em um projeto mais antigo, e que neste caso, detectou duas vulnerabilidades:

![Desktop View]({{ "/assets/img/tutoriais/security/security013.jpg" | relative_url }})


## Instalando e usando a gem hakiri

Vamos dar uma rápida olhada na gem hakiri, como disse acima, ela verifica por vulnerabilidades no sistema operacional, linguagens e outras ferramentas instalada na máquina, servidor ou container onde rodamos o projeto.

Para instalá-la, também não é necessário adicionar a gem no Gemfile, porém ela instala algumas gems que são dependências para que ela possa funcionar corretamente.

Rode os comandos abaixo: (na pasta do projeto)

```Terminal
$gem install hakiri
```
![Desktop View]({{ "/assets/img/tutoriais/security/security008.jpg" | relative_url }})

A gem estando instalada, basta gerar o manifesto json e após rodar o scan propriamente dito:

```Terminal
$hakiri manifest:generate
```
![Desktop View]({{ "/assets/img/tutoriais/security/security009.jpg" | relative_url }})


Rodando o scan:

```Terminal
$hakiri system:scan
```
![Desktop View]({{ "/assets/img/tutoriais/security/security010.jpg" | relative_url }})


## Conferindo o relatório gerado pelo hakiri

Como podemos ver na imagem acima, o hakiri não detectou muitas das ferramentas geralmente instaladas em servidores que rodam aplicações rails, afinal, estou rodando na minha máquina local, um WSL 2 rodando no Windows 10. Porém, conforme podemos ver na outra imagem, ele detectou diversas vulnerabilidades no sistema operacional, e isto é importante, pois precisamos estar atentos não só a segurança da nossa aplicação, mas também a segurança da nossa infra, seja ela local ou na nuvem.

![Desktop View]({{ "/assets/img/tutoriais/security/security011.jpg" | relative_url }})


## Rodei os scanners! Meu Deus, e agora?!

Como podemos ver, estas são duas gems bem completas, e que fazem um bom trabalho. Sugiro que verifiquem as documentações e testem exaustivamente em seus sistemas. Existem outras ferramentas, porém ainda não testei, e assim que testar, volto aqui e atualizo.

Mas, e agora? O que vem depois?

Bom, o relatório gerado pela gem brakeman é bem detalhado, mostrando extamente em que linha do seu código está a possível vulnerabilidade, e como ela pode ser explorada. Neste caso, você pode seguir as boas práticas e refatorar, conforme indicado na documentação de boas práticas de segurança do rails, neste link:

> Securing Rails Applications - v7.0.0: <https://guides.rubyonrails.org/security.html>

Já, no caso da gem hakiri, ela dá um relatório mais detalhado sobre as vulnerabilidades do sistema, e sendo assim, o melhor caminho é instalar os patches de segurança recomendados, e caso você não tenha permissão para executar certas tarefas, contate o administrador do sistema. Você terá feito um bom serviço.


## Finalizando...

Bom, foi um tutorial bem simples, e direto ao ponto, e confesso que este pouco material me ajudou muito quando precisei checar a segurança das minhas aplicações RoR. Espero que seja útil para vocês também. 

É isso pessoal!

Um ótimo final de 2021, e que 2022 venha com tudo, trazendo o que há de bom e do melhor para nós e nossas famílias!

Grande Abraço.

MadSantana! :-) 




