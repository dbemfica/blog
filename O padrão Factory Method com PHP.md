## Introdução
Dando continuiedade a serie sobre os padrões de projeto(designer patterns) com a linguagem PHP. Se você não sabe o que eu estou falando, este é o segundo artigo sobre uma serie que eu vou escrevendo. Caso você queria ler os outros artigos dessa serie pode acessar os links abaixo.

[SOLID com PHP](https://diogobemfica.com.br/solid-com-php) (prologo)

[O padrão Singleton com PHP](https://diogobemfica.com.br/o-padrao-singleton-com-php) (primeiro artigo)

Neste artigos vou tentar passar a minha interpretação baseado nos meus estudos de como funcione e como deve ser implementado o padrão Factory Method. E vamos implementar este padrão com a linguagem PHP.

## Factory Method
O Factory Method é um padrão de projeto de criação. Ele define uma interface para a criação de objetos. Acho importante deixar claro que quando estou falando de interface não esou falando somente de criar uma *interface* literalmente. Uma classe Abstrata também pode ser considerada um interface neste caso. Tudo vai depender do que se encaixa melhor para a usa implementação.

Diferente do padrão Abstratic Method o Factory Method é mais flexsivel, deixando para as subclasses mais epecializadas para a criação desses objetos. Vamos intender isso no decorrer do artigo.

### Problema a resolver
Vamos imaginar que você foi contratado para desenvolver um API para prover as informações de diversos smtartphones de diversas fabricantes.