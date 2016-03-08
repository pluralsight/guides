O [Jekyll][jekyll] é um gerador de páginas estáticas. A ideia do Jekyll é permitir que você crie páginas estáticas utilizando apenas marcação com HTML e Markdown. O sistema utiliza o padrão de template [Liquid][liquid] e formato [YAML][yaml] para armazenar variáveis e facilitar a reutilizo de código.

Quando começamos a produção de um site, geralmente separamos as partes do layout (header, footer e etc) e incluímos essas partes nas páginas onde for preciso evitando a repetição de código. Grande parte das pessoas (inclusive eu) escolhem algum [CMS][cms] (Wordpress, Drupal etc) e usam os includes pra resolver essa parte do problema. 

### E qual é o problema disso?
O primeiro problema é que, nem sempre é possível entregar pro cliente (em um freela por exemplo), pacotes com arquivos `.php` ou qualquer outra extensão para arquivos de linguagens server-side. Pode ser que seu cliente não tenha um servidor preparado para hospedar suas páginas desenvolvidas utilizando aquela sua ferramenta tão querida.

Segundo, mesmo que seja para um projeto pessoal (um blog por exemplo), se você quiser ter uma maior liberdade e customizar temas, usar plugins do Wordpress, os recursos oferecidos no [wordpress.com][wordpress] provávelmente não vão ser o suficiente pra você customizar tudo "na mão" e aí, você vai ter usar a versão instalada do Wordpress.
A versão instalada do Wordpress é pesada. Se você parar pra medir a quantidade de chamadas ao banco de dados a cada requisição, vai ver que eu não estou mentindo...

Fora isso, escrever seu post tendo que se preocupar com a formatação é bem chato e falando particularmente, usar [markdown][markdown] pra escrever é bem mais elegante.

> Não estou falando pra você parar de usar o wordpress ou seu CMS preferido no seu blog, pra trabalhar ou o que quer que seja. Se ele te atende, use!

Chega de falar e vamos ver como o Jekyll funciona na prática. Nesse post vou tentar explicar em um passo a passo bem simples com o exemplo de um blog com o Jekyll.

### Baby steps do Jekyll


### Instalação
O Jekyll foi escrito em [ruby][ruby-link], ele é uma gem (o que são [ruby gems][ruby-gems]?), então vamos precisar do ruby instalado pra começar a usa-lo.

Para instalar o ruby:

```
 ~ $ sudo apt-get install ruby
```

> Se você usa o windows, o [@fnando][nando-twitter] tem um [post][nando-post] bem completo explicando a instalação do ruby no windows.

Tendo o ruby instalado, podemos instalar a gem do Jekyll:

```
~ $ gem install jekyll
```

> Sobre problemas na instalação, a [documentação][jekyll-install] é bem completa e com certeza vai te ajudar.


Pronto! Aqui já temos o Jekyll já está instalado e pronto para ser usado.


### Iniciando um projeto
Após instalarmos o Jekyll, podemos iniciar um novo projeto em qualquer diretório:

```
 ~ $ jekyll new my_blog
```

Dessa forma será criada a seguinte estrutura de arquivos:

![Estrutura de arquivos](https://raw.githubusercontent.com/andreybleme/andreybleme.github.io/master/assets/img/estrutura_de_arquivos.png "Estrutura de arquivos")

Todos os arquivos com um *_ underline* na frente do nome serão ignorados no pacote final gerado pelo Jekyll.

- Na pasta `_includes` são colocados os elementos de página que se repetem (header, footer etc).

- Na pasta `_layouts` ficam os padrões de layout das páginas, voce pode criar vários de acordo com a necessidade de se ter diferentes layouts para grupos de páginas.

> Os layouts e includes normalmente usam variáveis YAML como `page.title` para evitar a repetição de conteúdo. [Entenda melhor +][yaml-doc]

- Em `_posts` ficam seus arquivos `.markdown` com o conteúdo dos seus posts. O Jekyll usa por padrão nomes do tipo: *2015-29-01-nome-do-post.markdown*, sendo o nome e a data contidas no arquivo usados na construção da URL.

- O arquivo `_config.yml` contém as informação referentes ao site, essas informações são lidas sempre que o Jekyll é iniciado. 

- Quando o build do projeto for feito pelo Jekyll, o pacote final se encontra na pasta `_site`. É lá que está o projeto pronto para ser publicado. 


### Criando um post
Para criar um novo post, basta adicionar um novo arquivo na pasta `_posts` com o nome seguindo os padrões do Jekyll:

```
2015-29-01-bem-vindo-ao-jekyll.markdown
```

Ao editar o arquivo, podemos digitar nosso post normalmente utilizando a [sintaxe markdown][markdown-sintaxe] para formatar o conteúdo do post. 

O Jekyll vai transformar o arquivo em HTML e adicionar ao pacote do projeto junto ao layout, especificado no cabeçalho do arquivo do post.

O Jekyll ainda adiciona pastas no pacote `_site` automaticamente de acordo com as categorias especificadas em `categories` uma variavel no cabeçalho do arquivo do post, coisa fina. Seguindo o exemplo do blog, isso faria com que a URL ficasse assim: 

```
meublog.com/blog/jekyll/2015/01/27/bem-vindo-ao-jekyll.html
```


### Deploy
Agora que já temos nosso post, queremos fazer o deploy e publicar!

Podemos fazer build do pacote final no mesmo diretório do projeto, onde o Jekyll gera o pacote final na pasta `_site`:

```
~ $ jekyll build
```

Ou podemos fazer build e gerar o pacote final em outro diretório:

```
~ $ jekyll build --destination /home/projects/
```

Assim teremos nosso pacote final, pronto para ser publicado.


### Integração com Github Pages 
Uma coisa muito legal do Jekyll, é que com ele você pode hospedar suas páginas gratuitamente (pra sempre) com o serviço do Github, o [github pages][git-pages]. 
Pra usar o serviço, você precisa ter um repositório remoto no github com o nome: `username.github.io` onde *username* é o seu nome de usuário no github.

Tendo feito isso,  acesse seu repositório local onde se encontram os arquivos do seu projeto Jekyll (se o diretório ainda não for um repositório git, basta executar `git init`), e sincronize o repositório local com seu repositório remoto:

```
git remote add origin https://github.com/username/username.github.io.git
```

Depois, adicionamos nossos arquivos do repositório local ao remoto:

```
git push -u origin master
```

Agora sempre que você alterar seu repositório local e commitar as alterações ao repositório remoto do github, as alterações vão ser feitas e seu projeto vai estar acessível ao mundo todo pelo seu endereço `username.github.io`!

> Esse "workflow" ainda contribui para que haja entregas bem mais contínuas nos seus projetos \o/


Por enquanto é isso. Até a próxima :) 




[jekyll]:      		http://jekyllrb.com
[liquid]: 	   		http://liquidmarkup.org/
[yaml]:		   		http://pt.wikipedia.org/wiki/YAML
[cms]: 		   		http://pt.wikipedia.org/wiki/Sistema_de_gerenciamento_de_conte%C3%BAdo
[wordpress]:   		http://wordpress.com
[markdown]:    		https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#code
[ruby-link]: 		http://pt.wikipedia.org/wiki/Ruby_%28linguagem_de_programa%C3%A7%C3%A3o%29
[nando-post]: 		http://simplesideias.com.br/configurando-ruby-rails-mysql-e-git-no-windows
[nando-twitter]: 	https://twitter.com/fnando
[ruby-gems]: 		https://rubygems.org/
[jekyll-install]: 	http://jekyllrb.com/docs/installation/
[markdown-sintaxe]: https://github.com/adam-p/markdown-here/wiki/Markdown-Here-Cheatsheet
[yaml-doc]: 		http://jekyllrb.com/docs/frontmatter/
[git-pages]: 		https://pages.github.com/