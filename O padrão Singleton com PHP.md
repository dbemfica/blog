## Introdução
Esse é para ser o primeiro de uma série de artigos que vou escrever para documentar os meus estudos sobre os designer pattern(Padrões de Projetos) e aproveitar para compartilhar esses estudos com a comunidade em forma de artigos.

Nessa série vou demonstrar alguns dos principais padrões de projeto do mercado. A minha ideia não é falar de todos os padrões existentes, mas sim explicar os principais deles e demonstrar com exemplo a sua implementação na linguagem PHP.

## Singleton Pattern
Para começarmos achei melhor falar sobre o padrão Singleton. Ele um padrão simples que prega a ideia de uma instância única a onde ao tentar criar uma nova instância da classe ela detecta que essa instância já existe e retorna ela mesma não deixando que se crie uma nova. Esse padrão é facilmente encontrado em classes que fazem a conexão com o banco de dados ou uma classe responsável pelos logs de uma aplicação.

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

Como podemos ver no exemplo acima a cada nova chamada da classe *Log* nós criamos uma nova instancia. começando com *#1* e ir até *#5*
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