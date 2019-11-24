Hoje vou publicar a parte 2 da série de artigos sobre o RabbitMQ, se você caiu de paraquedas nesse artigo eu recomendo fortemente que você acesse o primeiro artigo desse seria para não ficar perdido no assunto. O primeiro artigo foi bastante teórico mais muito importante se você nunca ouviu falar ou sabe muito ponto sobre o RabbitMQ. Você pode acessar o primeiro artigo clicando neste link [Introdução ao RabbitMQ](https://diogobemfica.com.br/Introducao-ao-rabbitmq).

No artigo desta semana vou explicar sobre o tipo de Exchange **direct** e vou mostrar como usar o PHP para fazermos tanto a publicação quanto o consumo das mensagens.

## Exchange direct
Uma Exchange do tipo direct é talvez o tipo mais que existe. Basicamente temos uma Exchange e uma fila de mesmo nome e ao enviar uma mensagem para a Exchange vai para fica de mesmo nome. Exemplos uma Exchange com o nome de **enviar_email** vamos ter uma fila(queue) com o mesmo nome **enviar_email** se você enviar uma mensagem para esta Exchange a mensagem vai para fila **enviar_email**. Simples assim.

No RabbitMQ já vem com uma Exchange desse tipo configurada por padrão nome dela é **(AMQP default)**. Essa Exchange server como Exchange padrão do sistema, então se nenhuma Exchange for informada no hora de enviar a mensagem para o RabbitMQ essa Exchange que será acionada.

### Routing key
Para entendermos como a Exchange **(AMQP default)** funcionado preciso explicar o conceito de Routing key. Uma **Routing key** (chave de roteamento) é uma chave que a Exchange pode usar para tomar a decisão de qual fila enviar a mensagem. No caso da Exchange **(AMQP default)** você não informa a Exchange mais deve informar uma Routing key. Assim está Exchange vai enviar a mensagem para fila de mesmo nome da **Routing key**.

## Publicando uma mensagem.
Agora que entendemos o básico podemos finalmente partir para a implementação usando o PHP. Primeiro de tudo você precisa se certificar que o seu PHP possui as seguintes extensões **ext-bcmath**, **ext-sockets**. Instalar essas extensões é uma tarefa nada complicada, mas caso precisa de alguma ajuda, deixe um comentário que farei o possível para ajudar.

Com as extensões instaladas vamos instalar a biblioteca mais utilizada para trabalhar com AMQP que é o protocolo utilizado pelo RabbitMQ. Para fazer isso rode o seguinte comando.

```shell
composer require php-amqplib/php-amqplib
```

Agora com biblioteca instalada, vamos criar um arquivo será o responsável por enviar as mensagem para o RabbitMQ. Vamos ele de **sender.php**. Dentro desse arquivos vamos começar importantando com as classes que vamos precisar.

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;
```

Depois vamos precisar iniciar a comunicação passando quatro parâmetros o **host** a **porta** o **usuário** e a **senha**. Depois disso precisar estâncias um canal que será usado para enviar as mensagens.
```php
$host = 'localhost';
$porta = 5672;
$usuario = 'guest';
$senha = 'guest';
$connection = new AMQPStreamConnection($host, $porta, $usuario, $senha);
$channel = $connection->channel();
```

Agora que temos o nosso canal vamos declarar uma fila com o nome **minha_fila** que caso ela não existe ela será criada no RabbitMQ.
```php
$channel->queue_declare('minha_fila');
```
 > Você pode criar as filas previamente, se a fila já existir esse comando não é mais necessário. Isso também fará com que comunição com o RabbitMQ aconteça de forma bem mais rápida.

Vamos criar a nova mensagem que vamos enviar para a nossa nova fila.
```php
$conteudo = 'primeira mensagem';
$msg = new AMQPMessage($conteudo);
```

E por fim vamos fazer a publicação, vamos precisar informar uma **Exchange** vazia e uma **Routing key** com o mesmo nome da fila que acabamos de criar.

```php
$exchange = '';
$routingKey = 'minha_fila';
$channel->basic_publish($msg, '', $routingKey);
echo "Mensagem enviada: '" . $conteudo . "'\n";
```

Para finalizar precisamos fechar a o canal e a conexão.
```php
$channel->close();
$connection->close();
```

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

$channel->queue_declare('minha_fila');

$conteudo = 'primeira mensagem';
$msg = new AMQPMessage($conteudo);

$exchange = '';
$routingKey = 'minha_fila';
$channel->basic_publish($msg, '', $routingKey);
echo "Mensagem enviada: '" . $conteudo . "'\n";

$channel->close();
$connection->close();
```

Se executarmos ***php sender.php*** no terminal veremos a seguinte mensagem.
```shell
[dbemfica@dbemfica-pc rabbitmq-php]$ php sender.php
Mensagem enviada: 'primeira mensagem'
```
Também podemos acessar o RabbitMQ no endereço **http://localhost:15672/#/queues** e podemos ver que a nossa mensagem foi enviada.

![Primeira mensagem](https://diogobemfica.com.br/multimidia/2019_11_24_rabbitmq_first_message.png)

## Consumindo uma mensagem
Agora que conseguimos enviar mensagens para o RabbitMQ está na hora de sabermos como podemos fazer o consumo destas mensagem. O processo e bem semelhante ao anterior vamos criar um aquivo **consumer.php** e dentro desse arquivo importar as classes que vamos precisar.

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
```

Depois vamos precisar iniciar a comunicação da mesma forma que fizemos no processo anterior.
```php
$host = 'localhost';
$porta = 5672;
$usuario = 'guest';
$senha = 'guest';
$connection = new AMQPStreamConnection($host, $porta, $usuario, $senha);
$channel = $connection->channel();
```

Também declaramos a fila para caso ela não exista.
```php
$channel->queue_declare('minha_fila');
```
 > Sempre lembrando que esse processo é opcional

Agora as coisas mudam um pouco, vamos precisar criar uma função de callback de será responsável por processar a nossa mensagem. Neste caso vamos simplesmente imprimi-la na tela.
```php
$callback = function ($msg) {
    echo "Mensagem recebida '" . $msg->body . "'\n";
};
```

Para fazer o consumo usamos a função **basic_consume** e esta função pode receber um monte de parâmetros. Mas neste momento acho importante se atermos ao exemplo abaixo a onde basicamente passamos o nome da fila e a função callback e criamos anteriormente.
```php
$queue = 'minha_fila';
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
 > Nos próximos artigos eu explicarem cada um dos parâmetros que a função **basic_consume** recebe.

 Para finalizar precisamos fechar a o canal e a conexão como fizemos anteriormente.
```php
$channel->close();
$connection->close();
```

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

$channel->queue_declare('minha_fila');

$callback = function ($msg) {
    echo "Mensagem recebida '" . $msg->body . "'\n";
};

$queue = 'minha_fila';
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

Se executarmos ***php consumer.php*** no terminal veremos a seguinte mensagem.
```shell
[dbemfica@dbemfica-pc rabbitmq-php]$ php consumer.php
Mensagem recebida 'primeira mensagem'
```
E também podemos acessar o RabbitMQ no endereço **http://localhost:15672/#/queues** e podemos ver que a nossa mensagem não está mais na fila.
![Consumimimdo a primeira mensagem](https://diogobemfica.com.br/multimidia/2019_11_24_rabbitmq_consume_first_message.png)

Estes estes são os primeiros passos para entrarmos no mundo da mensageria, espero que esteja gostando, lembre-se que sempre precisar por deixar um comentário aqui no blog teria prazer em ajudar.

No artigo da semana que vem vou falar sobre o Tipo de Exchange **fanout**, espero ver você semana que vem. até lá.

## Referências

https://www.cloudamqp.com/blog/2015-05-18-part1-rabbitmq-for-beginners-what-is-rabbitmq.html

https://www.rabbitmq.com/tutorials/tutorial-one-php.html

https://www.youtube.com/watch?v=RnIwwm00UPM

