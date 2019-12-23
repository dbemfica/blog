Sejá bem vindo ao penúltimo artigo desta série sobre o RabbitMQ. Muito obrigado pela sua presença e por ter chegado até aqui nessa jornada comigo sobre o mundo da mensageria e se você é novo aqui eu recomendo que você visite este link [Artigos sobre mensageria](https://diogobemfica.com.br/categoria/mensageria). para ter acesso desde de o primeiro artigo desta série e poder tirar o melhor proveito dos conteúdo abordados aqui.

Hoje vou falar sobre o último tipo de Exchange oferecido pelo RabbitMQ, estou falando do Exchange **headers**. Sem dúvida o tipo mais complexo que podemos usar e também o que mais oferece flexibilidade para trabalhar nas nossas rotas. E como não podia deixar de ser vamos usar o PHP para trabalharmos com esse tipo de Exchange.

## Exchange headers
Em uma Exchange do tipo **headers** diferente os tipo **direct** e **topic** não usamos a **Routing keys** para definirmos o roteamento das nossas mensagens. Nesse tipo como o nome sugere nós usamos o header(cabeçalhos) das mensagem para definirmos as regras do roteamento. Basicamente o header enviado nas mensagem precisa combinar com o bind(ligação) feita entre a fila e a Exchange. Vamos ao exemplo para ajudar a entender melhor esse conceito, nós vamos ligar uma Exchange a uma fila passando o seguintes argumentos *['format' => 'pdf', 'type' => 'file']* Nestes argumentos definimos duas chaves, uma *format* com o valor *pdf* e outra chave *type* com o valor *file*. Agora para conseguirmos enviar uma mensagem para a nossa fila precisamos informar no header da mensagem essas chaves exatamente com estes valores, se não a nossa mensagem não será entregue.

Existe uma chave especial no RabbitMQ para conseguirmos definir algumas regras no nosso roteamento, esta chave é a **x-match** Esta chave tem dois valores possíveis. Os valores são:

 * x-match = all - Todos os valores no header precisar combinar(padrão).
 * x-match = any - Pelo menos um valor no header precisar combinar.

Voltando ao exemplo anterior se modificarmos os argumentos passados na hora de fazer a ligação da Exchange com a fila para *['format' => 'pdf', 'type' => 'file', 'x-match' => 'any']* a regra muda. Agora se passarmos *'format' => 'pdf'* ou *'type' => 'file'* a mensagem vai ser enviada para fila já que pelo menos uma das chaves vai combinar.

## Na prática.
Agora que entendemos como funciona uma Exchange do tipo **headers** vamos partir para o nosso exemplo prático. Vamos criar três filas: **imgArchiver** que vai armazenar qualquer tipo de imagem; **imgJpg** que vai fazer todo o processamento das imagem do tipo JPG e **imgPng** que vai fazer todo o processamento das imagem do tipo PNG. Nossa arquitetura de filas vai ficar da seguinte forma.
![Esquema Exchange headers](https://diogobemfica.com.br/multimidia/2019_12_22_exchange_headers.jpg)

Vamos implementar o exemplo acima. Como sempre vamos criar um arquivo chamado **sender.php** e vamos fazer conexão com o RabbitMQ. Mas desta vez vamos precisar importar uma nova classe chamada **AMQPTable** ela será muito importante para trabalharmos com os argumentos e headers das mensagens. Vejamos um exemplo abaixo como vai ficar o nosso código.

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;
use PhpAmqpLib\Wire\AMQPTable;

$host = 'localhost';
$porta = 5672;
$usuario = 'guest';
$senha = 'guest';
$connection = new AMQPStreamConnection($host, $porta, $usuario, $senha);
$channel = $connection->channel();
```

Agora vamos usar o método do **exchange_declare** para criar a nossa Exchange **logs** do tipo **topic**.
```php
$nomeExchange = 'imagem';
$tipoExchange = 'headers';
$channel->exchange_declare($nomeExchange, $tipoExchange);
```

Agora vamos usar o método **queue_declare** para criar as três filas.

```php
$channel->queue_declare('imgArchiver');
$channel->queue_declare('imgJpg');
$channel->queue_declare('imgPng');
```

Agora vamos fazer a ligação das nossas três filas com a Exchange, vamos usar a classe **AMQPTable** para criar um objeto **$arguments** que vai conter as nossas regras de roteamento. Preste atenção nos valores passados para cada ligação.

```php
$queue = 'imgArchiver';
$exchange = 'imagem';
$routingKey = '';
$nowait = false;
$arguments = new AMQPTable(['type' => 'img', 'x-match' => 'any']);
$channel->queue_bind($queue, $exchange, $routingKey, $nowait, $arguments);

$queue = 'imgJpg';
$exchange = 'imagem';
$routingKey = '';
$nowait = false;
$arguments = new AMQPTable(['type' => 'img', 'ext' => 'jpg']);
$channel->queue_bind($queue, $exchange, $routingKey, $nowait, $arguments);

$queue = 'imgPng';
$exchange = 'imagem';
$routingKey = '';
$nowait = false;
$arguments = new AMQPTable(['type' => 'img', 'ext' => 'png']);
$channel->queue_bind($queue, $exchange, $routingKey, $nowait, $arguments);
```

Agora vamos enviar a nossa "imagem" do tipo JPG para a Exchange. Mais uma vez usamos a classe **AMQPTable** mas desta vez montamos um objeto **$headers** e usamos o método **set** do objeto **$msg** passando o objeto **$headers** como valor de uma chave chamada **application_headers**. Assim a nossa mensagem passa a possuir as chaves *type* e *ext* com os seus respectivos valores.

```php
//IMAGEM JPG
$conteudo = 'conteudo em base64 da imagem jpg';
$msg = new AMQPMessage($conteudo);

$headers = new AMQPTable(['type' => 'img', 'ext' => 'jpg']);
$msg->set('application_headers', $headers);
```
Depois disso basta fazer a publicação.

```php
$exchange = 'imagem';
$channel->basic_publish($msg, $exchange);
echo "Mensagem enviada: '" . $conteudo . "'\n";
```

Antes de executar o nosso código vamos fazer o mesmo processo para uma "imagem" do tipo PNG, o processo a basicamente o mesmo e o resultado vai ficar da seguinte forma.

```php
//IMAGEM PNG
$conteudo = 'conteudo em base64 da imagem png';
$msg = new AMQPMessage($conteudo);

$headers = new AMQPTable(['type' => 'img', 'ext' => 'png']);
$msg->set('application_headers', $headers);

$exchange = 'imagem';
$channel->basic_publish($msg, $exchange);
echo "Mensagem enviada: '" . $conteudo . "'\n";
```

Fechamos a conexão.
```php
$channel->close();
$connection->close();
```

E o resultado final no nosso arquivo **sender.php** é o seguinte.
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;
use PhpAmqpLib\Wire\AMQPTable;

$host = 'localhost';
$porta = 5672;
$usuario = 'guest';
$senha = 'guest';
$connection = new AMQPStreamConnection($host, $porta, $usuario, $senha);
$channel = $connection->channel();

$nomeExchange = 'imagem';
$tipoExchange = 'headers';
$channel->exchange_declare($nomeExchange, $tipoExchange);

$channel->queue_declare('imgArchiver');
$channel->queue_declare('imgJpg');
$channel->queue_declare('imgPng');

$queue = 'imgArchiver';
$exchange = 'imagem';
$routingKey = '';
$nowait = false;
$arguments = new AMQPTable(['type' => 'img', 'x-match' => 'any']);
$channel->queue_bind($queue, $exchange, $routingKey, $nowait, $arguments);

$queue = 'imgJpg';
$exchange = 'imagem';
$routingKey = '';
$nowait = false;
$arguments = new AMQPTable(['type' => 'img', 'ext' => 'jpg']);
$channel->queue_bind($queue, $exchange, $routingKey, $nowait, $arguments);

$queue = 'imgPng';
$exchange = 'imagem';
$routingKey = '';
$nowait = false;
$arguments = new AMQPTable(['type' => 'img', 'ext' => 'png']);
$channel->queue_bind($queue, $exchange, $routingKey, $nowait, $arguments);

//IMAGEM JPG
$conteudo = 'conteudo em base64 da imagem jpg';
$msg = new AMQPMessage($conteudo);

$headers = new AMQPTable(['type' => 'img', 'ext' => 'jpg']);
$msg->set('application_headers', $headers);

$exchange = 'imagem';
$channel->basic_publish($msg, $exchange);
echo "Mensagem enviada: '" . $conteudo . "'\n";

//IMAGEM PNG
$conteudo = 'conteudo em base64 da imagem png';
$msg = new AMQPMessage($conteudo);

$headers = new AMQPTable(['type' => 'img', 'ext' => 'png']);
$msg->set('application_headers', $headers);

$exchange = 'imagem';
$channel->basic_publish($msg, $exchange);
echo "Mensagem enviada: '" . $conteudo . "'\n";

$channel->close();
$connection->close();
```

Agora se executarmos o nosso código vamos poder ver como ficou as nossas fila no RabbitMQ.
![Mensagem enviada para fila para filas de imagens](https://diogobemfica.com.br/multimidia/2019_12_22_sender_exchange_tipe_headers.png)

Como podemos ver a fila **imgArchiver** recebeu as duas mensagens já ela possui a regra de *['type' => 'img', 'x-match' => 'any']* fazendo com que batesse enviar no header a mensagem o *'type' => 'img'* para que a mensagem fosse enviada para ela.

Da mesmo forma como no artigo anterior não vou dar exemplos de como fazer o consumo destas filas já que o processo é mesmo demonstrados nos primeiros artigos da série. Se está acompanhando não terá dificuldades. Mas em caso de dúvidas sinta-se a vontade para fazer qualquer pergunta nos comentários, terei prazer em ajudar.

Uma Exchange do tipo **headers** é menos usada do que as demais tipos mas é a mais complexa e flexível que o RabbitMQ oferece. Espero que esse pequeno exemplo possa ter te ajudado e entender o seu funcionamento e te instigue a pesquisar mais sobre as possibilidades deste tipo de Exchange.

Estamos chegando no final da série e na semana que vem teremos o último artigo com aspectos mais teóricos do RabbitMQ que são muitos importantes e que você vai precisar entender para tirar melhor proveito desde **Message Broker**. Espero ver você na semana que vem. Abraços.

## Referências

https://www.rabbitmq.com/tutorials/amqp-concepts.html

https://github.com/php-amqplib/php-amqplib/tree/master/demo

https://www.tutlane.com/tutorial/rabbitmq/csharp-rabbitmq-headers-exchange
