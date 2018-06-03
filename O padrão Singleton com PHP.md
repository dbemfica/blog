## Introdução
Esse é para ser o primeiro de uma série de artigos que vou escrever para documentar os meus estudos sobre os designer pattern(Padrões de Projeto) e aproveitar para compartilhar esses estudos com a comunidade em forma de artigos.

Nessa série vou demonstrar alguns dos principais padrões de projeto do mercado. A minha ideia não é falar de todos os padrões existentes, mas sim explicar alguns dos principais e demonstrar com um exemplo a sua implementação na linguagem PHP.

## Design Patterns
Antes de partirmos para o nosso primeiro padrão como esse é o primeiro artigo dessa série. Achei melhor dar um pequena explicação sobre o que são os padrões de projeto de uma forma mais ampla e contar um pouco da sua história.

Para começar nada melhor do citar um dos livros de maior referência sobre o assunto, estou falando do "Padrões de Projeto - Soluções Reutilizáveis de Software Orientado a Objetos" da famosa Gang of Four(Gangue dos Quatro) de 1994. A onde os quatro autores "Erich Gamma", "John Vlissides", "Ralph Johnson" e "Richard Helm" descrevem 24 design pattern. Eles os separam em 3 categorias: padrões de criação, padrões estruturais e padrões comportamentais. Nessa série eu vou escolher dois padrões de cada categoria para explicar e demonstrar o seu funcionamento.

Apesar existirem mais do que os 24 padrões mencionados, este livro mesmo nos dias atuais como mencionado anteriormente é uma das maiores referências sobre o assunto.

## Singleton Pattern
Agora podemos começar, e para isso achei melhor falar sobre o padrão de criação Singleton. Vou explicar a sua estrutura, qual o problema ele busca resolver e mostrar um exemplo com um como podemos usar esse padrão nos nossos projetos desenvolvidos com o PHP.

### Problema a resolver
O padrão Singleton prega a ideia da instância única. A onde quando você tenta criar uma instância da sua classe, essa classe deve verificar a existência da instância para que se a instância não existir criar uma nova e caso contrário retornar a mesma instância sem criar uma nova. Assim garantindo menor consumo de memória e um único ponto de acesso global para esse recurso.

O Singleton tem uma estrutura bem simples e de fácil compreensão. como mostrado na imagem abaixo.

![Diagrama UML de uma classe singleton.](https://raw.githubusercontent.com/webfatorial/PadroesDeProjetoPHP/master/Creational/Singleton/uml/uml.png)
 > Esse é o diagrama UML do padrão Singleton


Esse padrão é bastante usado para o desenvolvimentos de classes responsáveis pelo o gerenciamento da conexão com o banco de dados ou o gerenciamentos de logs. Nós permitindo ter acesso a esses recursos na nossa aplicação de forma rápida e descomplicada.

### Implementação normal
Agora vamos finalmente falar de código. Mas primeiro vamos criar um classe da forma convencional, uma pequena classe de Logs. Vou usar esse exemplo demonstra que a cada instância criada estamos literalmente criando uma nova instância na memória.

```php
<?php
class Log
{
}

$log = new Log;
$log2 = new Log;
$log3 = new Log;
$log4 = new Log;
$log5 = new Log;

var_dump($log);
var_dump($log2);
var_dump($log3);
var_dump($log4);
var_dump($log5);
```
Se executarmos o exemplo acima podemos ver que a cada nova chamada da classe *Log* nós criamos uma nova instância. começando com *#1* e indo até *#5*
```bash
object(Log)#1 (0) {
}
object(Log)#2 (0) {
}
object(Log)#3 (0) {
}
object(Log)#4 (0) {
}
object(Log)#5 (0) {
}
```

### Implementação Singleton
Agora vamos começar a implementar o padrão Singleton na nossa classe, vamos começar a fazer isso alterando a visibilidade dos nosso métodos mágicos *__construct* o *__wakeup* e do *__clone*.

```php
private function __construct()
{
}

private function __clone()
{
}

private function __wakeup()
{
}
```
Com isso garantimos que a classe não pode ser mais instanciada de forma normal.


Vamos criar uma propriedade privada estática *$instance*
```php
private static $instance;
```
 E também vamos criar um método estático *getInstance* que vai ser o único responsável por retornar a nova instância da nossa classe. É neste método que vamos verificar se já existe uma instância na nossa classe.

```php
public static function getInstance()
{
    if(self::$instance === null){
        self::$instance = new self;
    }
    return self::$instance;
}
```

Se der certo vamos ter um arquivo como no exemplo abaixo.
```php
<?php
class Log
{
    private static $instance;

    private function __construct()
    {
    }

    private function __clone()
    {
    }

    private function __wakeup()
    {
    }

    public static function getInstance()
    {
        if(self::$instance === null){
            self::$instance = new self;
        }
        return self::$instance;
    }
}
```

Agora com a nossa classe devidamente modificada podemos testar o nosso exemplo
```php
$log = Log::getInstance();
$log2 = Log::getInstance();
$log3 = Log::getInstance();
$log4 = Log::getInstance();
$log5 = Log::getInstance();

var_dump($log);
var_dump($log2);
var_dump($log3);
var_dump($log4);
var_dump($log5);
```
No exemplo acima agora com o nosso Singleton funcionando vamos ver que a nossa instância é sempre a mesma *#1*. Conseguimos garantir a mesma instância não importa quantas vezes chamamos a nossa classe.
```bash
object(Log)#1 (0) {
}
object(Log)#1 (0) {
}
object(Log)#1 (0) {
}
object(Log)#1 (0) {
}
object(Log)#1 (0) {
}
```

## Singleton é um anti pattern?
Para muitos desenvolvedores o Singleton é considerado um anti patterns, mas por que será? Acredito que uns dos principais motivos é pela natureza estática. Isso nos traz alguns problemas como não podermos trabalhar com interfaces e o aumento de acoplamento na classe. Além disso temos como estamos falando de acesso global, existe um perigo de sobrescrever o comportamento da classe e gerar comportamentos inesperados no sistema.

Acredito que esses dois são os principais motivos para que o Singleton tenha esse estigma. Mas na minha opinião ele é um padrão como qualquer outro, tem suas vantagens e desvantagens cabe ao desenvolvedor ter a sabedorias para saber quando e onde usá-lo.

## Conclusão
Como falei anteriormente esse artigo é primeiro de uma série que pretendendo escrever para compartilhar os meus estudos com a comunidade. Comecei falando do Singleton por ser um padrão bastante famoso e de fácil compreensão. Espero que com esses pequenos passos que possa ter de demostrado como podemos facilmente criar a nossa classe usando o Singleton no PHP.

## Referências
<https://www.schoolofnet.com/curso-design-patterns-pt2-padroes-de-criacao/>
<https://www.casadocodigo.com.br/products/livro-design-paterns-php>
<https://github.com/webfatorial/PadroesDeProjetoPHP/tree/master/Creational/Singleton>
