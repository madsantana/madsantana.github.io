---
title: Subindo uma Loja Virtual Solidus no Heroku com Postgres e Cloudinary
author: Madsantana, aka, Marco Antonio Damaceno
date: 2020-07-23 07:11 +0300
categories: [Blogging]
tags: [tutoriais, tutorial, rauby, rails, heroku, cloudinary, solidus, solidusshop, deploy, devops]
pin: true
---
![Ruby Badge](https://img.shields.io/badge/-Ruby-red)
![Rails Badge](https://img.shields.io/badge/-Rails-green)
![Heroku Badge](https://img.shields.io/badge/-Heroku-purple)
![Postgres Badge](https://img.shields.io/badge/-Postgres-grey)
![Cludinary Badge](https://img.shields.io/badge/-Cloudinary-blue)

Eae pessoal! Tudo bem com vocês?!
<div style="text-align: justify">
A uns dias atrás, reproduzi na minha VM Vagrant, um tutorial bem legal do Sérgio Lima, do Onebitcode, ensinando a subir uma loja com a gem Solidus. Porém o tutorial ensinava o básico do básico, e uma modificação simples. Mas, foi
o suficiente para despertar a curiosidade neste que vos escreve.

Encontrei um outro tutorial, este da gringa, ensinando a subir uma loja Solidus no Heroku, usando Sqlite e armazenando
as imagens numa instância S3 da AWS. Até aí, tudo bem, mas e quando você não tem um tostão para gastar com hospedagem 
e armazenamento?

E foi aí que surgiu a idéia deste tutorial, onde procuro usando as bases dos dois tutorias citados acima, mostrar como podemos, sem gastar um tostão, subir uma loja virtual funcional, usando Ruby on Rails, a gem solidus, armazenando as imagens no Cloudinary e fazendo o deploy no Heroku. Ou seja, ao final deste tutorial, você terá uma loja virtual solidus totalmente funcional, ficando à sua escolha a customização da mesma.

Abaixo, seguem os links dos tutoriais originais:</div>

> Tutorial do Sérgio Lima (Onebitcode): <https://onebitcode.com/e-commerce-rails/>

> Tutorial na Gringa (em inglês): <https://nebulab.it/blog/deploy-solidus-to-heroku-like-a-boss>

## Bora por a mão na massa?!

<div style="text_align: justify">
O que precisaremos para tocar este projetinho?
Lembrando que pressuponho aqui, que você já tem o heroku-cli instalado no seu ambiente. Caso não tenha, nos dois tutoriais acima é ensinado como instalar.
</div>

|Ferramenta|Versão|Observação|
|:---|:--|---:|
|Ruby | 2.6.3 | - |
|Rails | 5.1.4 | - |
|PostgreSQL | 10 | - |
|Gem Solidus | - | - |
|Cloudinary | - | É necessário cadastro |
|Heroku / Heroku Cli | - | É necessário Cadastro |

Vamos preparar o ambiente e criar o projeto, utilizando o Rails, para isso, execute os passos a seguir:

```Terminal
$mkdir my_ecommerce01
$cd my_ecommerce01
$rvm use ruby-2.5.0@solidusshop --ruby-version --create
$gem install rails -v 5.1.4
$rails _5.1.4_ new . --database=postgresql
```

Inicialize o respositório do git com os comandos abaixo:

```Terminal
$git init
$git add .
```

Faça um commit iniciai para o projeto:

```Terminal
$git commit -m "Initial commit on project..."
```

Para tornar o seu projeto uma loja virtual simples, porém funcional, instale a gem Solidus.
Adicione as seguintes linhas ao seu Gemfile:

```Console
gem 'solidus'
gem 'solidus_auth_devise'
gem 'cloudinary'
gem 'carrierwave'
gem 'solidus_cloudinary'
gem 'deface'
```

O seu Gemfile deve se parecer, com algumas variações, com o exemplo da imagem abaixo:

![Desktop View]({{ "/assets/img/tutoriais/solidus-heroku/tutorial-006.JPG" | relative_url }})

**Configure a aplicação para acessar o postgresql, modificando de acordo, o arquivo config/database.yml, vide imagem:**

![Desktop View]({{ "/assets/img/tutoriais/solidus-heroku/tutorial-006-A.JPG" | relative_url }})

Instale as gems e suas dependências,rodando:

```Terminal
$bundle update
$bundle install
```

Crie o banco de dados, instale e rode as migrations com os seguintes comandos:

```Terminal
$rake db:create
$rails g spree:install
```

**Caso você ainda não tenha criado o Banco de Dados com rake db:create, o rails g spree:install, roda um script que executará a criação do banco de dados, fará as migrations, e preencherá este banco de dados, com dados de exemplo.**

**Neste processo, será solicitado a criação de um usuário administrador para a loja (usuário - email, e senha).**

Configurando a gem de autenticação:

```Terminal
$rails g solidus:auth:install
```

Instala as novas migrations:

```Terminal
$rake railties:install:migrations
```

Executa as novas migrações:

```Terminal
$rake db:migrate
```

Agora você já pode testar sua aplicação para ver se está tudo ok. Para isso, rode o comando abaixo, e acesse no seu 
browser o seguinte endereço: http://localhost:3000

```Terminal
$rails s
```

![Desktop View]({{ "/assets/img/tutoriais/solidus-heroku/tutorial-011.JPG" | relative_url }})

Se tudo ocorreu bem até aqui, você deve vizualisar algo como a imagem abaixo:

![Desktop View]({{ "/assets/img/tutoriais/solidus-heroku/tutorial-012.JPG" | relative_url }})

Parabéns! Sua loja está funcionando!

Agora, pressupondo que você já tem as contas no Cloudinary e Heroku, e já tem as credenciais necessárias, vamos
fazer o deploy e colocar sua loja em produção, no Heroku.

## Configurando o Heroku

A partir daqui, assumimos que você já tem o heroku-cli instalado. Caso não tenha, no tutotial do onebitcode citado lá no começo, é ensinado como instalar.

Execute no seu terminal o comando para criar uma aplicação no Heroku:

```Terminal
$heroku apps:create myecommerce01
```
<div style="text_align: justify">
Este comando cria uma nova aplicação no Heroku chamada "myecommerce01", porém como ela já existe, pode ser que dê erro, neste caso, execute o comando sem nomear a aplicação ou, escolha um nome diferente. Será criado também um reposítório, que será adicionado ao seu arquivo .git/config, e você pode checar isso com o comando:</div>

```Terminal
$git remote -v
```

Configure o ambiente de produção no Heroku, instalando o PostgreSQL:

```Terminal
$heroku addons:create heroku-postgresql
```

Após executar esta instalação, pegue a URL do banco de dados criado, com o comando:

```Terminal
$heroku config
```

O comando acima, deve retornar algo como abaixo:

```Terminal
=== my_ecommerce Config Vars
DATABASE_URL: postgres://XXXX:yyyyy@ec2-1-2-3-4.compute.amazonaws.com:5432/zzzzz
```

**A variável DATABASE_URL aponta para a localização do seu banco de dados. Agora voce pode adicionar no seu arquivo config/database.yml abaixo das configurações de produção.**

```Terminal
production:
  <<: *default
  url: <%= ENV['DATABASE_URL'] %>
```

## Configurando o Cloudinary

Pronto. Agora precisamos configurar nossa aplicação para armazenar as imagens dos produtos no Cloudinary, uma vez que o Heroku, assim que é feito o deploy, apaga todas as imagens e não permite o armazenamento local.

De posse das suas credenciais do Cloudinary (cloudname, api_key e api_secret_key), execute os passos abaixo:

**Crie um initializer para o Cloudinary (config/initializers/solidus_cloudinary.rb)**

```Terminal
$touch config/initializers/solidus_cloudinary.rb
```

E adicione a seguinte configuração ao arquivo criado:

```Terminal
Cloudinary.config do |config|
  config.cloud_name = 'sample'
  config.api_key = '874837483274837'
  config.api_secret = 'a676b67565c6767a6767d6767f676fe1'
  config.cdn_subdomain = true
end
```

Antes de fazer o deploy para o Heroku, é necessário modificar o modo como o javascript faz a compressão, atualize o seguinte arquivo: config/environments/production.rb

Modifique a linha:

```Terminal
config.assets.js_compressor = :uglifier 
```
com:

```Terminal
config.assets.js_compressor = Uglifier.new(harmony: true) - Figura 019
```

## Fazendo o deploy da sua aplicação no Heroku.

Agora, faça um commit com todas as alterações recentes e execute o push do seu código para o Heroku:

```Terminal
$git add .
$git commit -m "Configure heroku Environment..."
$git push heroku master
```

Se o push foi executado sem erros, ao final do comando, você deve obter uma resposta como a da imagem abaixo:

![Desktop View]({{ "/assets/img/tutoriais/solidus-heroku/tutorial-020.JPG" | relative_url }})

Agora você deve popular seu banco de dados com dados de teste, para isso existe o comando heroku run, que executará algumas tarefas extras após o deploy da sua aplicação:

Faça a migration d banco de dados:

```Terminal
$heroku run rake db:migrate
```

Caso você não queira popular a loja, já online, com dados de exemplo, ela está pronta, basta cadastrar seus produtos e vender, ou, através de outras configurações, gems, ou tutoriais, fazer novas customizações, como mudar o idioma, logotipo, layout, etc.

No caso de ser apenas uma loja teste, prossiga como abaixo:

Popule o banco de dados com as informações de teste:

```Terminal
heroku run rake db:seed
```

Carregue os dados de exemplo com:

```Terminal
$heroku run rake spree_sample:load
```

A operação de seed criará o usuário admin, e a configuração básica para iniciar seu eCommerce e permitir que você comece a vender na internet.

Agora você pode ver sua aplicação rodando em: https://myecommerce01.herokuapp.com (ou outro nome que você tenha escolhido) ou usando o comando:

```Terminal
$heroku open
```

Caso ocorra algum problema ao acessar a loja no Heroku, você pode checar os logs com o comando:

```Terminal
$heroku logs
```

Enfim chegamos ao final deste primeiro tutorial aqui da página, e se tudo ocorreu corretamente, você deve visualizar sua página no ar, como abaixo:

![Desktop View]({{ "/assets/img/tutoriais/solidus-heroku/tutorial-025.JPG" | relative_url }})

E no caso de não ter utilizado os dados de exemplo, será visualizada como abaixo:

![Desktop View]({{ "/assets/img/tutoriais/solidus-heroku/tutorial-022.JPG" | relative_url }})

Dá uma olhada na loja criada neste tutorial:

> Loja criada para este tutorial: <https://myecommerce01.herokuapp.com/>


## Conclusão

É isso pessoal, este foi meu primeiro tutorial, e espero de coração que vocês tenham gostado, e que seja tão útil para vocês como foi últil para mim ao escrevê-lo.

Gostou?Tem críticas, correções ou mesmo sugestões de novos tópicos, por favor, curta, comente, e compartilhe... este espaço é todo seu!

Obrigado!

## Referências

> Tutorial do Sérgio Lima (Onebitcode): <https://onebitcode.com/e-commerce-rails>

> Tutorial na Gringa (em inglês): <https://nebulab.it/blog/deploy-solidus-to-heroku-like-a-boss>

> Heroku: <https://heroku.com>

> Cloudinary: <https://cloudinary.com>

> Lista de Comandos Heroku: <https://devcenter.heroku.com/articles/heroku-cli-commands>
