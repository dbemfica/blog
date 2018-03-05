## Introdução
Esse é para ser o primeiro de uma série de artigos que vou escrever para documentar os meus estudos sobre os designer pattern(Padrões de Projetos) e aproveitar para compartilhar esses estudos com a comunidade em forma de artigos.

Nessa série vou demonstrar alguns dos principais padrões de projeto do mercado. A minha ideia não é falar de todos os padrões existentes, mas sim explicar os principais deles e demonstrar com exemplo a sua implementação na linguagem PHP.

## Designer Paterns
Antes de partirmos para o nosso primeiro padrão como esse é o primeiro artigo dessa seria. Achei melhor dar um pequena explicação de onde vem e o que são os designer patterns.

De acordo com o livro "Padrões de Projeto - Soluções Reutilizaveis de Software Orientado a Objetos" da famosa Gang of Four(A gangue dos quatro) de 1994. Os quatro autores "Erich Gamma", "John Vlissides", "Ralph Johnson" e "Richard Helm" descrevem 24 designers patterns seperados em 3 categorias: padroes de criação, estruturais, comportamentais.

Apesar de existirem mais do que os 24 padroes mecionadas no livro as 3 categorias são uma das maiores referencias quando se fala nesse assunto.

## Singleton Pattern
Agora podemos começar, e para isso achei melhor falar sobre o padrão Singleton. Vou explicar a sua estrutura, qual o problema ele busca resolver e mostrar um exemplo com o nosso querido PHP.


## Problema a resolver
O padrão Singleton cria uma estrutura para que a sua clase só possa ser extanciada uma unica vez no tempo de execução do seu projeto e garantir o acesso global a mesma.

Para isso ele usa a seguinte estrutura.

![Diagrama UML de uma classe singleton.](https://pt.wikipedia.org/wiki/Singleton#/media/File:Singleton.png)

Comecei essa serie falando do Singleton por ser um padrão bastante famoso e de facil copreenção. Com pequenos passos podemos facilmente criar a nossa classe usando o Singleton. E para exemplificarmos vamos simular uma classe para o gerenciamento de logs da nossa aplicação.

Primeiro vamos criar um classe da forma normal. Vou usar esse exemplo demostrar a instancia unica da nossa classe.

```php
<?php
class Log
{
    public function __construct()
    {
    }
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

Como podemos ver no exemplo acima a cada nova chamada da classe *Log* nós criamos uma nova instancia. começando com *#1* e indo até *#5*
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

Agora vamos implementar o padrão Singleton na nossa classe, vamos começar a fazer isso alterando a visibilidade dos nosso metodos magicos *__construct* o *__wake* e do *__clone*.

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

Com isso garantimos que a classe não pode ser mais instanciada.

Vamos criar uma propriedade privada $instance e vamos criar um metodo *getInstance* que vai ser o unico responsavel por retornar a nova instancia da nossa classe. Nesse metodo vamos verificar se a instancia já foi istanciada.

```php
public static function getInstance()
{
    if(self::$instance === null){
        self::$instance = new self;
    }
    return self::$instance;
}
```


Se der certo vamos ter um arquivo como no exemplo a baixo.
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
No exemplo acima agora com o nosso Singleton funcionando vamos ver que a nossa as nossas instancia é sempre a mesma *#1*. Conseguimos garantir a mesma instancia.
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