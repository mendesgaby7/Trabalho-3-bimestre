# Validação de dados e requisições

## Introdução

A criação de uma API HTTP exige algumas medidas que garantam a armazenagem de dados de forma íntegra e segura, seguindo um padrão de negociação predefinido. Para isso, podemos utilizar diferentes ferramentas; no nosso tutorial faremos o uso do **yup**, um construtor de esquemas e validador de dados.

Além disso, utilizaremos estruturas já existentes no JavaScript para demais verificações.

## API HTTP
A API que vamos usar de base para a validação de dados é uma API de livros, onde cada livro vai ter os campos "id", "nome", "autor" e "ano". Segue o exemplo:

| Livro 1        | Livro 2          
| ------------- |-------------| 
| 1     | 2 |
| Harry Potter e a Pedra Filosofal     | Duna      |
| J. K. Rowling | Frank Herbert     |
| 1997 | 1965|

> **É importante que notemos o formato de cada um desses campos.** No caso, o campo "ano" se trata de um número e "autor" e "nome" trabalham
> no formato de string. Já o "id" é o único campo não manipulável pelo usuário, sendo auto incrementado pelo próprio banco de dados. 

O usuário vai ter a possibilidade de consultar cinco tipos de requisições. São elas:

 - **listarLivros** - Uma requisição GET que lista todos os livros cadastrados no banco de dados
 - **novoLivro** - Uma requisição POST que cadastra um novo livro no banco de dados
 - **obterLivro** - Uma requisição GET que busca e exibe um livro já cadastrado
 - **editarLivro** - Uma requisição PUT que edita um livro já cadastrado
 - **apagarLivro** - Uma requisição DELETE que apaga um livro do banco de dados

## yup 
Considerando a pré-instalação dos pacotes para a criação e integração do nosso banco de dados, precisamos instalar aquele que faremos uso somente para o processo de validação (no nosso caso, o **yup**)

    npm install -S yup
Em seguida, podemos chamar o pacote no nosso controlador.

    const  yup  =  require('yup');

Como já citado, esse validador trabalha por meio de esquemas. O usuário monta um método personalizado de aprovação para seus dados e é retornado *true* ou *false*, dependendo do resultado da checagem.
![alt text](https://imgur.com/tGhmpYJ.png)


Tendo as ferramentas à disposição e sabendo o que cada requisição pede, começamos o processo de validação.

## Trabalhando com as validações
Nossa primeira tarefa é em cima da **listarLivros**. Nessa requisição, precisamos garantir que haja uma verificação da quantidade de livros para, em seguida, exibir ao usuário.

![alt text](https://imgur.com/AptoO8o.png)

Após requerer a lista inteira de livros cadastrados no banco de dados, uma coisa é certa: essa lista terá algum ou nenhum livro. Seguindo esse raciocínio, só precisamos verificar qual o tamanho da lista com um *if else*. Caso não haja nenhum livro, vamos exibir o erro 404 junto de uma mensagem; se houver, exibimos a lista.
***
Com a **novoLivro**, precisamos implementar o yup. Primeiramente guardaremos o corpo da requisição, suas chaves e as chaves do nosso modelo de negociação em diferentes constantes. Também precisaremos de uma variável para guardar o status da consulta de chaves.

![alt text](https://imgur.com/3mqrnwn.png)

O processo segue da seguinte forma:

 ![alt text](https://imgur.com/0DLxXH2.png)

 1. Verificamos se os campos que o usuário digitou na requisição são compatíveis com a nossa proposta de modelo ("nome", "autor" e "ano). Para isso faremos o uso de um *for*, onde ocorre a comparação das chaves do corpo da requisição e as chaves do nosso modelo de negócio. Caso sejam incompatíveis, nossa variável de status indica *true* e força o loop a parar, indicando um erro 400 de sintaxe. Já caso sejam compatíveis, a variável indica *false* e o processo continua.
   
 ![alt text](https://imgur.com/OoX5gqo.png)
 
 2. Fazemos a validação de cada campo utilizando o yup. Primeiro criamos uma constante para armazenar nosso esquema de validação, onde indicamos que o nossos campos "nome" e "autor" são strings obrigatórias e o nosso campo "ano" deve ser um número inteiro e positivo. Caso nossa checagem retorne *false*, exibimos o erro 400 junto de uma mensagem. 
 
 ![alt text](https://imgur.com/m94TyNT.png)
 
 3. Caso passe em ambas as etapas, o corpo da requisição é adicionado ao banco de dados.
***
A requisição **obterLivro** é, de certa forma, parecida com a listarLivros. É interessante observar que caso o usuário forneça um id inexistente (ou seja, um espaço que não existe no banco de dados), é retornado um array vazio, Dessa forma, podemos trabalhar com a propriedade *length* desse array.

 ![alt text](https://imgur.com/tdnYjGq.png)

 Guardamos o tamanho do livro com o id fornecido numa constante e comparamos: se o objeto que foi devolvido pelo banco de dados for igual a zero, retornamos o erro 404 junto de uma mensagem personalizada. Caso contrário, exibimos o livro.
***
Na **editarLivro** teremos uma espécie de mescla entre as requisições *obterLivro* e *novoLivro*, sendo talvez a "mais trabalhosa". Precisamos verificar se a requisição respeita as regras de negócio, verificar se existem livros cadastrados, examinar se o livro indicado pelo id foi de fato cadastrado, validar os novos dados e por fim injetar as alterações no banco.

 ![alt text](https://imgur.com/kZ92OK3.png)

 1. A estrutura aqui é a mesma. Criaremos as constantes das chaves para comparação e a variável de status; em seguida, verificamos se são compatíveis. Caso não sejam, é exibido o erro 400 e uma mensagem.

 ![alt text](https://imgur.com/FjRpZAI.png)

 2. Caso sejam, a próxima etapa é verificar se existem livros no nosso banco de dados. Caso o tamanho indicado pela constante *livros* seja igual a 0, é exibido o erro 400 junto de uma mensagem personalizada. Caso contrário, partimos para a validação dos dados.

![alt text](https://imgur.com/thd9d8G.png)

 3.  Agora precisamos examinar se o livro indicado pelo id existe no nosso banco de dados. Fazemos isso como na *obterLivro*, verificando o tamanho do objeto salvo numa variável.

 ![alt text](https://imgur.com/GVuq6SJ.png)

 4. Os dados são validados da mesma forma que na *novoLivro*. Caso retorne *false*, é sinalizado um erro e uma mensagem. Já caso retorne *true*, os dados do corpo da requisição substituem os do banco. 

***
Por fim, trabalharemos a requisição **apagarLivro**. A princípio, ela se parece muito com a *listarLivros*, no entanto, além de verificar a quantidade de livros, também precisamos avaliar se o livro de fato existe no banco de dados para que ocorra a deleção.

 ![alt text](https://imgur.com/tk4Y1mb.png)

 Caso não hajam livros cadastrados, é exibido o erro 404 com uma mensagem. 

 ![alt text](https://imgur.com/klPZvPC.png)

 Se houverem, ocorre a verificação do tamanho do objeto. Caso o livro exista naquele id, ele é deletado do banco de dados.


## Referências
https://javascript.plainenglish.io/validate-your-node-express-js-rest-api-calls-with-yup-5c4080fdae87?gi=f695cc2c577f

https://www.w3schools.com/js/js_objects.asp
https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Object/keys

https://www.npmjs.com/package/yup

https://bradhick.medium.com/yup-valida%C3%A7%C3%B5es-no-react-de-uma-forma-muito-simples-700c039114e3