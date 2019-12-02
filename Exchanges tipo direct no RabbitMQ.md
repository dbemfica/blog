Hoje vou publicar a parte 3 da série de artigos sobre o RabbitMQ, se você caiu de paraquedas nesse artigo eu recomendo fortemente que você acesse os artigos anteriores desta série para não ficar perdido no assunto. Você pode acessar os outros artigos dessa série clicando neste link [Artigos sobre mensageria](https://diogobemfica.com.br/categoria/mensageria).

No artigo desta semana vou explicar sobre o tipo de Exchange **direct** e vou mostrar como usar o PHP para fazermos tanto a publicação quanto o consumo das mensagens.

## Exchange direct
Uma Exchange do tipo **direct** é um Exchange guiada pela **Routing key**. Está Exchange é ligada a quantos filas você quiser e usamos o **Routing key** para definir para qual fila a mensagem será enviada. Vejamos a imagem abaixo para entender melhor.

![Esquema Exchange direct](https://diogobemfica.com.br/multimidia/2019_12_01_esquema_exchange_direct.jpg)

Nessa imagem nós temos uma Exchange chamada **emitirNfe** do tipo **direct** ligada a três filas diferentes: **enviaParaReceita**, **salvaXml** e **enviaEmailCliente**. Você pode estar se perguntando mas se enviarmos uma mensagem para essa Exchange nós não estaríamos enviando a mensagem para as três filas ao mesmo tempo? É aí que entra a **Routing key**, na hora de enviarmos(publicar) a mensagem para essa Exchange enviamos também a **Routing key** e usamos ela para definir para qual fila deve ir a mensagem.

Vamos dar uma exemplo. Olhando na imagem acima quando queremos enviar uma mensagem somente para fila **enviarParaReceita** nós precisamos enviar junto a **Routing key** **enviar** assim a nossa mensagem não vai parar nas outras filas.

## Publicando uma mensagem.
Agora que entendemos como esse tipo de Exchange funciona podemos seguir para a implementação usando o PHP. Vamos fazer como fizemos na imagem acima. Vamos criar uma Exchange do tipo **direct** chamada **emitirNfe**.

Primeiro vamos criar um arquivo chamado **sender.php** e vamos fazer conexão com o RabbitMQ. Vejamos um exemplo abaixo

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

Agora vamos usar o método do **exchange_declare** para criar a nossa Exchange, nós passamos como primeiro parâmetro o nome da Exchange e como segundo o seu tipo.
```php
$nomeExchange = 'emitirNfe';
$tipoExchange = 'direct';
$channel->exchange_declare($nomeExchange, $tipoExchange);
```
 > Como mencionado no artigo anterior esses métodos que usamos para declarar uma Exchange ou uma fila são opcionais se a Exchange ou a fila já existirem no RabbitMQ não precisam ser usados.

Agora vamos usar o método **queue_declare** visto no artigo anterior para criar as três filas **enviaParaReceita**, **salvaXml** e **enviaEmailCliente**.

```php
$channel->queue_declare('enviaParaReceita');
$channel->queue_declare('salvaXml');
$channel->queue_declare('enviaEmailCliente');
```

Agora precisamos fazer a ligação das filas com a Exchange, para isso usamos o método **queue_bind** passando três parâmetros, primeiro o nome da fila, segundo o nome da Exchange e terceiro a **Routing key**. Este processo precisa ser feito para as três filas.

```php
$queue = 'enviaParaReceita';
$exchange = 'emitirNfe';
$routingKey = 'enviar';
$channel->queue_bind($queue, $exchange, $routingKey);

$queue = 'salvaXml';
$exchange = 'emitirNfe';
$routingKey = 'salvar';
$channel->queue_bind($queue, $exchange, $routingKey);

$queue = 'enviaEmailCliente';
$exchange = 'emitirNfe';
$routingKey = 'email';
$channel->queue_bind($queue, $exchange, $routingKey);
```

Pronto agora quando quisermos enviar uma mensagem para uma fila especificamente usados as **Routing key**. Vejamos um exemplo abaixo.

```php
$conteudo = 'conteudo da XML para receita federal';
$msg = new AMQPMessage($conteudo);

$exchange = 'emitirNfe';
$routingKey = 'enviar';
$channel->basic_publish($msg, $exchange, $routingKey);
echo "Mensagem enviada: '" . $conteudo . "'\n";

$channel->close();
$connection->close();
```

Se executarmos ***php sender.php*** no terminal veremos a seguinte mensagem.
```shell
[dbemfica@dbemfica-pc rabbitmq-php]$ php sender.php
Mensagem enviada: 'conteúdo da XML para receita federal'
```
E vamos poder ver no RabbitMQ que somente a fila **enviaParaReceita** recebeu a mensagem. As outras estão zeradas.

![Mensagem enviada para fila enviaParaReceita](https://diogobemfica.com.br/multimidia/2019_12_01_sender_exchange_tipy_direct_1.png)

Vamos comentar o envio anterior e criar um novo mas agora vamos enviar a mensagem para fila **salvaXml**

```php
$conteudo = 'conteudo da XML para receita federal';
$msg = new AMQPMessage($conteudo);

// $exchange = 'emitirNfe';
// $routingKey = 'enviar';
// $channel->basic_publish($msg, $exchange, $routingKey);
// echo "Mensagem enviada: '" . $conteudo . "'\n";

$exchange = 'emitirNfe';
$routingKey = 'salvar';
$channel->basic_publish($msg, $exchange, $routingKey);
echo "Mensagem enviada: '" . $conteudo . "'\n";

$channel->close();
$connection->close();
```
 > Comentamos o código anterior que fez a publicação na fila e criamos um novo igual ao anterior mas mudamos somente a **Routing key**.

Se executarmos o **sender.php** agora vamos ter a mesma saída no terminal e vamos poder ver que adicionamos uma mensagem na fila **salvaXml** e as outras filas continuaram os mesmos números de mensagens.

![Mensagem enviada para fila salvaXml](https://diogobemfica.com.br/multimidia/2019_12_01_sender_exchange_tipy_direct_2.png)

E para finalizar o processo de publicação não vamos deixar de fora da fila **enviaEmailCliente**, vamos exatamente o mesmo processo anterior.

```php
$conteudo = 'conteudo da XML para receita federal';
$msg = new AMQPMessage($conteudo);

// $exchange = 'emitirNfe';
// $routingKey = 'enviar';
// $channel->basic_publish($msg, $exchange, $routingKey);
// echo "Mensagem enviada: '" . $conteudo . "'\n";

// $exchange = 'emitirNfe';
// $routingKey = 'salvar';
// $channel->basic_publish($msg, $exchange, $routingKey);
// echo "Mensagem enviada: '" . $conteudo . "'\n";

$exchange = 'emitirNfe';
$routingKey = 'email';
$channel->basic_publish($msg, $exchange, $routingKey);
echo "Mensagem enviada: '" . $conteudo . "'\n";

$channel->close();
$connection->close();
```

Como podemos ver na imagem abaixo a fila **enviaEmailCliente** recebeu sua mensagem enquanto das outras continuaram intactas.

![Mensagem enviada para fila enviaEmailCliente](https://diogobemfica.com.br/multimidia/2019_12_01_sender_exchange_tipy_direct_3.png)

No final o nosso arquivo **sender.php** vai ter o seguinte conteúdo.
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

$nomeExchange = 'emitirNfe';
$tipoExchange = 'direct';
$channel->exchange_declare($nomeExchange, $tipoExchange);

$channel->queue_declare('enviaParaReceita');
$channel->queue_declare('salvaXml');
$channel->queue_declare('enviaEmailCliente');

$queue = 'enviaParaReceita';
$exchange = 'emitirNfe';
$routingKey = 'enviar';
$channel->queue_bind($queue, $exchange, $routingKey);

$queue = 'salvaXml';
$exchange = 'emitirNfe';
$routingKey = 'salvar';
$channel->queue_bind($queue, $exchange, $routingKey);

$queue = 'enviaEmailCliente';
$exchange = 'emitirNfe';
$routingKey = 'email';
$channel->queue_bind($queue, $exchange, $routingKey);

$conteudo = 'conteudo da XML para receita federal';
$msg = new AMQPMessage($conteudo);

// $exchange = 'emitirNfe';
// $routingKey = 'enviar';
// $channel->basic_publish($msg, $exchange, $routingKey);
// echo "Mensagem enviada: '" . $conteudo . "'\n";

// $exchange = 'emitirNfe';
// $routingKey = 'salvar';
// $channel->basic_publish($msg, $exchange, $routingKey);
// echo "Mensagem enviada: '" . $conteudo . "'\n";

$exchange = 'emitirNfe';
$routingKey = 'email';
$channel->basic_publish($msg, $exchange, $routingKey);
echo "Mensagem enviada: '" . $conteudo . "'\n";

$channel->close();
$connection->close();
```

## Consumindo uma mensagem
Para fazer o consumo das mensagens enviadas para as três filas o processo basicamente o mesmo do artigo anterior, por isso não vou dar muitos detalhes desta vez.

Vamos precisar de uma arquivo, vamos chamar ele de **consumer.php**, vamos importar as classes necessárias e fazer a conexão com o RabbitMQ.

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;

$host = 'localhost';
$porta = 5672;
$usuario = 'guest';
$senha = 'guest';
$connection = new AMQPStreamConnection($host, $porta, $usuario, $senha);
$channel = $connection->channel();
```
Também declaramos as filas para caso elas não existam. Vamos declarar as filas que vamos trabalhar.
```php
$channel->queue_declare('enviaParaReceita');
$channel->queue_declare('salvaXml');
$channel->queue_declare('enviaEmailCliente');
```
 > Sempre lembrando que esse processo é opcional

Criamos o nosso **callback**.
```php
$callback = function ($msg) {
    echo "Mensagem recebida '" . $msg->body . "'\n";
};
```

E realizamos o consumo passando os parâmetros necessários e fechamos a conexão.
```php
$queue = 'enviaParaReceita';
$consumer_tag = '';
$no_local = false;
$no_ack = true;
$exclusive = false;
$nowait = false;
$channel->basic_consume($queue, $consumer_tag, $no_local, $no_ack, $exclusive, $nowait, $callback);
while ($channel->is_consuming()) {
    $channel->wait();
}

$channel->close();
$connection->close();
```
 > Lembrando que nos próximos artigos eu explicarei cada um dos parâmetros que a função **basic_consume** recebe.

Se executarmos ***php consumer.php*** no terminal veremos a seguinte mensagem.
```shell
[dbemfica@dbemfica-pc rabbitmq-php]$ php consumer.php
Mensagem recebida 'conteúdo da XML para receita federal'
```

E vamos pode ver se a mensagem sumiu da fila **enviaParaReceita**.

![Mensagem consumida da fila enviaEmailCliente](https://diogobemfica.com.br/multimidia/2019_12_01_sender_exchange_tipy_direct_4.png)

Agora vamos fazer o que fizemos para publicar as mensagens, vou comentar o consumo e criar um  para cada fila que temos e depois mostrar o resultado no RabbitMQ.
```php
// $queue = 'enviaParaReceita';
// $consumer_tag = '';
// $no_local = false;
// $no_ack = true;
// $exclusive = false;
// $nowait = false;
// $channel->basic_consume($queue, $consumer_tag, $no_local, $no_ack, $exclusive, $nowait, $callback);
// while ($channel->is_consuming()) {
//     $channel->wait();
// }

$queue = 'salvaXml';
$consumer_tag = '';
$no_local = false;
$no_ack = true;
$exclusive = false;
$nowait = false;
$channel->basic_consume($queue, $consumer_tag, $no_local, $no_ack, $exclusive, $nowait, $callback);
while ($channel->is_consuming()) {
    $channel->wait();
}
```
![Mensagem consumida da fila salvaXml](https://diogobemfica.com.br/multimidia/2019_12_01_sender_exchange_tipy_direct_5.png)

```php
// $queue = 'enviaParaReceita';
// $consumer_tag = '';
// $no_local = false;
// $no_ack = true;
// $exclusive = false;
// $nowait = false;
// $channel->basic_consume($queue, $consumer_tag, $no_local, $no_ack, $exclusive, $nowait, $callback);
// while ($channel->is_consuming()) {
//     $channel->wait();
// }

// $queue = 'salvaXml';
// $consumer_tag = '';
// $no_local = false;
// $no_ack = true;
// $exclusive = false;
// $nowait = false;
// $channel->basic_consume($queue, $consumer_tag, $no_local, $no_ack, $exclusive, $nowait, $callback);
// while ($channel->is_consuming()) {
//     $channel->wait();
// }

$queue = 'enviaEmailCliente';
$consumer_tag = '';
$no_local = false;
$no_ack = true;
$exclusive = false;
$nowait = false;
$channel->basic_consume($queue, $consumer_tag, $no_local, $no_ack, $exclusive, $nowait, $callback);
while ($channel->is_consuming()) {
    $channel->wait();
}
```

![Mensagem consumida da fila enviaEmailCliente](https://diogobemfica.com.br/multimidia/2019_12_01_sender_exchange_tipy_direct_5.png)

No final o nosso arquivo **consumer.php** vai ter o seguinte conteúdo.
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;

$host = 'localhost';
$porta = 5672;
$usuario = 'guest';
$senha = 'guest';
$connection = new AMQPStreamConnection($host, $porta, $usuario, $senha);
$channel = $connection->channel();

$channel->queue_declare('enviaParaReceita');
$channel->queue_declare('salvaXml');
$channel->queue_declare('enviaEmailCliente');

$callback = function ($msg) {
    echo "Mensagem recebida '" . $msg->body . "'\n";
};

// $queue = 'enviaParaReceita';
// $consumer_tag = '';
// $no_local = false;
// $no_ack = true;
// $exclusive = false;
// $nowait = false;
// $channel->basic_consume($queue, $consumer_tag, $no_local, $no_ack, $exclusive, $nowait, $callback);
// while ($channel->is_consuming()) {
//     $channel->wait();
// }

// $queue = 'salvaXml';
// $consumer_tag = '';
// $no_local = false;
// $no_ack = true;
// $exclusive = false;
// $nowait = false;
// $channel->basic_consume($queue, $consumer_tag, $no_local, $no_ack, $exclusive, $nowait, $callback);
// while ($channel->is_consuming()) {
//     $channel->wait();
// }

$queue = 'enviaEmailCliente';
$consumer_tag = '';
$no_local = false;
$no_ack = true;
$exclusive = false;
$nowait = false;
$channel->basic_consume($queue, $consumer_tag, $no_local, $no_ack, $exclusive, $nowait, $callback);
while ($channel->is_consuming()) {
    $channel->wait();
}

$channel->close();
$connection->close();
```

Espero que você tenha entendido o que são as Exchanges do tipo **direct** e que esteja gostando desta série de artigos. No artigo da semana que vem vou falar sobre o tipo de Exchange **fanout**, espero ver você semana que vem. até lá.

## Referências

https://www.cloudamqp.com/blog/2015-05-18-part1-rabbitmq-for-beginners-what-is-rabbitmq.html

https://www.rabbitmq.com/tutorials/tutorial-one-php.html

https://www.youtube.com/watch?v=RnIwwm00UPM
