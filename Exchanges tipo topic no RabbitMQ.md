Sejá bem vindo a parte 5 desta série de artigos sobre o RabbitMQ, se você está acompanhando desde do primeiro artigo já deixo aqui o meu muito obrigado e se você é novo aqui e não sabe nada sobre o RabbitMQ ou sobre esta série eu recomendo que você acessa os outros artigos para não ficar perdido já que algumas das suas dúvidas já podem ter sido respondidas nos artigos anteriores. Para ter acesso este conteúdo basta clicar no link [Artigos sobre mensageria](https://diogobemfica.com.br/categoria/mensageria).

No artigo de hoje vamos falar do tipo de Exchange que na minha opinião é o mais interessante, estou falando o tipo **topic**. Vou explicar o seu funcionamento e como sempre vou demonstrar como podemos usar esse tipo de Exchange usado o nosso amado PHP.

## Exchange topic
Semelhante a uma Exchange do tipo **direct** que usamos as **Routing keys** para fazermos o roteamento das mensagens enviadas, uma Exchange do tipo **topic** possui duas regrinhas que podem ser utilizadas, são caracteres especiais que se usado nas **Routing keys** deste tipo de Exchange podem nós ajudar bastante para fazermos o roteamento das mensagens. Esses caracteres são:

 * *(asterisco) para substituir uma palavra.
 * #(sustenido) para substituir zero ou mais palavras.

Vamos ao um exemplo para nós ajudarmos a entender melhor essas regras. Neste exemplo imaginamos que nós temos duas aplicações diferentes e vamos usar o RabbitMQ para enviarmos os logs dessas aplicações para outro lugar, um Elasticsearch por exemplo. Nas nossas aplicações nós possuímos três níveis de logs são eles: **info**, **warning** e **critical**. E na nossa aplicação temos a seguinte regra de negócio, todos os logs vão para suas filas específicas de cada aplicação e um log do nível **critical** também vai para uma fila especial que no nosso caso hipotético vai enviar um SMS para os desenvolvedores responsáveis. Já que se trata de bug crítico. Abaixo temos uma imagem para ilustrar a nossa arquitetura.

![Esquema Exchange topic](https://diogobemfica.com.br/multimidia/2019_12_15_exchange_topic.jpg)

## Na prática.
Vamos implementar o exemplo acima. Como sempre vamos criar um arquivo chamado **sender.php** e vamos fazer conexão com o RabbitMQ. Vejamos um exemplo abaixo.

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$host = 'localhost';
$porta = 5672;
$usuario = 'guest';
$senha = 'guest';
$connection = new AMQPStreamConnection($host, $porta, $usuario, $senha);
$channel = $connection->channel();
```

Agora vamos usar o método do **exchange_declare** para criar a nossa Exchange **logs** do tipo **topic**.
```php
$nomeExchange = 'logs';
$tipoExchange = 'topic';
$channel->exchange_declare($nomeExchange, $tipoExchange);
```
 > Mais uma vez vou mencionar que esses métodos que usamos para declarar uma Exchange ou uma fila são opcionais se a Exchange ou a fila já existirem no RabbitMQ não precisam ser usados.

Agora vamos usar o método **queue_declare** para criar as três filas **logsApp1**, **logsApp2** e **logsCritico**.

```php
$channel->queue_declare('logsApp1');
$channel->queue_declare('logsApp2');
$channel->queue_declare('logsCritico');
```
Vamos fazer a ligação das duas primeiras filas com a Exchange usando o método **queue_bind** e passando as nossas **Routing keys** exatamente como está na imagem.

```php
$queue = 'logsApp1';
$exchange = 'logs';
$routingKey = 'app1.*';
$channel->queue_bind($queue, $exchange, $routingKey);

$queue = 'logsApp2';
$exchange = 'logs';
$routingKey = 'app2.*';
$channel->queue_bind($queue, $exchange, $routingKey);
```
 > Como usamos o * no final nossa **Routing key** qualquer palavras que colocarmos no final vai ser aceita pela regra e vai para fila.

E para fazer a terceira ligação digitamos.
```php
$queue = 'logsCritico';
$exchange = 'logs';
$routingKey = '*.critical';
$channel->queue_bind($queue, $exchange, $routingKey);
```
 > Desta vez usamos o * no início nossa **Routing key** o quer dizer que não importa se for *app1* ou *app2* a mensagem vai ir para fila desde que o final seja igual a *.critical*.

Pronto agora vamos testar, vamos enviar uma mensagem para a fila **logsApp1** .

```php
$conteudo = 'Logs nivel info';
$msg = new AMQPMessage($conteudo);

$exchange = 'logs';
$routingKey = 'app1.info';
$channel->basic_publish($msg, $exchange, $routingKey);
echo "Mensagem enviada: '" . $conteudo . "'\n";

$channel->close();
$connection->close();
```
 Nós enviamos com a **Routing key** igual a **"app1.info"** lembrando que a nossa primeira regra diz para enviar os logs para **logsApp1** só precisamos informar **"app1."** e o final pode ser qualquer palavra já que usamos o *(asterisco).

Se executarmos o **sender.php** e acessarmos o RabbitMQ vamos poder ver a nossa mensagem está lá.
![Mensagem enviada para fila para fila logsApp1](https://diogobemfica.com.br/multimidia/2019_12_15_sender_exchange_tipy_topic_1.png)

Vamos comentar parte do código e implementar um envio para fila **logsApp2**.
```php
// $conteudo = 'Logs nivel info';
// $msg = new AMQPMessage($conteudo);

// $exchange = 'logs';
// $routingKey = 'app1.info';
// $channel->basic_publish($msg, $exchange, $routingKey);
// echo "Mensagem enviada: '" . $conteudo . "'\n";

$conteudo = 'Logs nivel warning';
$msg = new AMQPMessage($conteudo);

$exchange = 'logs';
$routingKey = 'app2.warning';
$channel->basic_publish($msg, $exchange, $routingKey);
echo "Mensagem enviada: '" . $conteudo . "'\n";

$channel->close();
$connection->close();
```
 Desta vez usamos a **Routing key** igual a **"app2.warning"** mas já que temos basicamente a mesma regra do caso anterior vai funcionar.

 Depois de executar o **sender.php** vamos conseguir ver a nossa mensagem foi enviada para a fila **logsApp2**
 ![Mensagem enviada para fila para fila logsApp2](https://diogobemfica.com.br/multimidia/2019_12_15_sender_exchange_tipy_topic_2.png)

 Agora para finalizar vamos enviar uma mensagem com um log nível crítico.
 ```php
// $conteudo = 'Logs nivel info';
// $msg = new AMQPMessage($conteudo);

// $exchange = 'logs';
// $routingKey = 'app1.info';
// $channel->basic_publish($msg, $exchange, $routingKey);
// echo "Mensagem enviada: '" . $conteudo . "'\n";

// $conteudo = 'Logs nivel warning';
// $msg = new AMQPMessage($conteudo);

// $exchange = 'logs';
// $routingKey = 'app2.warning';
// $channel->basic_publish($msg, $exchange, $routingKey);
// echo "Mensagem enviada: '" . $conteudo . "'\n";

$conteudo = 'Logs nivel critical';
$msg = new AMQPMessage($conteudo);

$exchange = 'logs';
$routingKey = 'app1.critical';
$channel->basic_publish($msg, $exchange, $routingKey);
echo "Mensagem enviada: '" . $conteudo . "'\n";

$channel->close();
$connection->close();
```
E como podemos ver a nossas regras funcionam perfeitamente, como nós tínhamos a regra de **"(asterisco).critical"** não importa se for o app1 ou app2 se a **Routing key** termina-se com **".critical"** a mensagem vai para fila **logsCritico**. Da forma de como tudo está arquitetado um log crítico tinha que ir para a fila **logsApp1** e um para a fila do log crítico(**logsCritico**).

Se executarmos o **sender.php** vamos poder ver que tu correu como planejado.

![Mensagem enviada para fila para fila logsCritico](https://diogobemfica.com.br/multimidia/2019_12_15_sender_exchange_tipy_topic_3.png)

E como de costume aqui está o como ficou o nosso arquivo **sender.php** depois de toda a implementação.

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$host = 'localhost';
$porta = 5672;
$usuario = 'guest';
$senha = 'guest';
$connection = new AMQPStreamConnection($host, $porta, $usuario, $senha);
$channel = $connection->channel();

$nomeExchange = 'logs';
$tipoExchange = 'topic';
$channel->exchange_declare($nomeExchange, $tipoExchange);

$channel->queue_declare('logsApp1');
$channel->queue_declare('logsApp2');
$channel->queue_declare('logsCritico');

$queue = 'logsApp1';
$exchange = 'logs';
$routingKey = 'app1.*';
$channel->queue_bind($queue, $exchange, $routingKey);

$queue = 'logsApp2';
$exchange = 'logs';
$routingKey = 'app2.*';
$channel->queue_bind($queue, $exchange, $routingKey);

$queue = 'logsCritico';
$exchange = 'logs';
$routingKey = '*.critical';
$channel->queue_bind($queue, $exchange, $routingKey);

// $conteudo = 'Logs nivel info';
// $msg = new AMQPMessage($conteudo);

// $exchange = 'logs';
// $routingKey = 'app1.info';
// $channel->basic_publish($msg, $exchange, $routingKey);
// echo "Mensagem enviada: '" . $conteudo . "'\n";

// $conteudo = 'Logs nivel warning';
// $msg = new AMQPMessage($conteudo);

// $exchange = 'logs';
// $routingKey = 'app2.warning';
// $channel->basic_publish($msg, $exchange, $routingKey);
// echo "Mensagem enviada: '" . $conteudo . "'\n";

$conteudo = 'Logs nivel critical';
$msg = new AMQPMessage($conteudo);

$exchange = 'logs';
$routingKey = 'app1.critical';
$channel->basic_publish($msg, $exchange, $routingKey);
echo "Mensagem enviada: '" . $conteudo . "'\n";

$channel->close();
$connection->close();
```

Desta vez não vou dar exemplos de como fazer o consumo das mensagem já que o processo é exatamente o mesmo dos artigos anteriores. Se você está acompanhado a série já está craque e não terá nenhuma dificuldade de realizar essa implementação.

Espero que esteja gostando desta série, se tiver qualquer duvida deixa um comentario de será um prazer ajuda-lo. Espero ver você semana que vem quando vamos falar do último tipo de Exchange, o tipo **headers**.

## Referências

https://www.cloudamqp.com/blog/2015-05-18-part1-rabbitmq-for-beginners-what-is-rabbitmq.html

https://www.rabbitmq.com/tutorials/tutorial-one-php.html

https://www.youtube.com/watch?v=RnIwwm00UPM
