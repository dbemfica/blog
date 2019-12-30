Sejá bem vindo ao último artigo dessa série que é uma introdução ao RabbitMQ. Desde já deixo aqui o meu muito obrigado para você que vem me acompanhando nessa jornada no mundo da mensageria. E para você que está vendo esse artigo e não sabe sobre o que eu estou falando acesse link [Artigos sobre mensageria](https://diogobemfica.com.br/categoria/mensageria) para poder ver desde de o primeiro artigo e não ficar perdido no assunto.

Neste último artigo vou explicar alguns conceitos que eu acho importante sobre o RabbitMQ e dar alguns detalhes das principais funções que temos usado nesta série. São coisas teóricas mas que podem lhe mostrar vários dos recursos que podemos usar com o RabbitMQ.

##Persistência
As mensagens por padrão no RabbitMQ ficam armazenadas na memória, isso nos garante uma velocidade muito boa para armazená-las e consumi las. Mais o lado ruim disso é que se o servidor do RabbitMQ for reiniciado as mensagens serão apagadas e as vezes dependendo da mensagem não podemos deixar isso acontecer. Então é possível dizer para o RabbitMQ persistir as nossas mensagens gravando elas no disco, isso garante que mesmo que o servidor reinicie as mensagens não serão apagadas, mas isso diminuir a performance do servidor. A cabe a você avaliar e tomar a melhor decisão.

##Durabilidade
Quando falamos de durabilidade no RabbitMQ o conceito é bastante semelhante do da percistentencia das mensagens. Só que dessa vez estamos falando das FIlas(Ques) e das Exchanges. Nós podemos definir que elas sejam duráveis fazendo com que caso o servidor seja reiniciado elas não sejam apagadas.

 > Aviso importante: Caso você não possa perder as mensagem dentro do RabbitMQ você precisar definir ela como persistente e as filas como duráveis. Não adianta nada só dizer que a mensagem é persistente se a fila for apagada.

##Acknowledge
No RabbitMQ temos o conceito do acknowledge, basicamente é dizer que uma mensagem pode ser apagada da fila depois dela ter sido consumida. Este conceito na minha opinião é um dos mais importantes, porque com ele caso acontecesse algum erro durante o consumo da mensagem podemos fazer com que a mensagem volta para fila de forma automática.

## Auto delete
Outro conceito muito interessante que devemos prestar atenção é o do auto-delete, com ele podemos fazer que com as filas sejam apagadas assim que não houver mais consumidores ligadas a ela. Este recurso é muito utilizado quando usamos o pattern como o RPC(Remote procedure call).

## Propriedades dos Métodos
Agora que entendemos este conceitos vou explicar as propriedades dos métodos que usamos durante este série nos nossos exemplo com a linguagem PHP.

### **exchange_declare**
* **exchange(string)** O nome que será dado para a Exchange.

* **type(string)** O tipo da Exchange podendo ser ele: **direct**, **fanout**, **headers** e **topic**.

* **passive(boolean)** Com este atributo definido como verdadeiro o servidor vai retornar um erro caso a Exchange não exista. Este atributo pode ser usado para checar a existência da Exchange sem modificar uma existente.

* **durable(boolean)** Com este atributo definimos se a nossa Exchange vai ser durável.

* **auto_delete(boolean)** Faz com que a exchange seja apagada quando todas filas ligadas são estejam mais sendo usadas.

* **internal(boolean)** Faz com que a Exchange não possa ser usada diretamente, mas somente através de outras exchanges.

* **nowait(boolean)** Ligando este atributo o método não vai esperar a resposta do servidor, seria como se trabalha-se de forma assíncrona não esperando a resposta dizendo que a Exchange foi criada.

* **arguments(AMQPTable)** Serve para passar uma tabela de argumentos opcionais para a Exchange.

### **queue_declare**
* **queue(string)** O nome que será dado para a fila.

* **passive(boolean)** Com este atributo definido como verdadeiro o servidor vai retornar um erro caso a fila não exista. Este atributo pode ser usado para checar a existência da fila sem modificar uma existente.

* **durable(boolean)** Com este atributo definimos se a nossa fila vai ser durável.

* **exclusive(boolean)** Definindo este atributo como verdadeiro a fila só poderá ser acessado por esta conexão e será deletada quando a conexão for fechada.

* **auto_delete(boolean)** Faz com quando não houver mais mensagens e consumidores conectados a fila será apagada.

* **nowait(boolean)** Ligando este atributo o método não vai esperar a resposta do servidor, seria como se trabalha-se de forma assíncrona não esperando a resposta dizendo que a fila foi criada.

* **arguments(AMQPTable)** Serve para passar uma tabela de argumentos opcionais para a fila.

### **queue_bind**
* **queue(string)** O nome da fila que será ligada a uma Exchange.

* **exchange(string)** O nome da Exchange que será ligada a uma fila.

* **routing_key(string)** A **Routing Key que será usado na ligação.

* **nowait(boolean)** Com este atributo ligado o método não vai esperar a resposta do servidor.

* **arguments(AMQPTable)** Serve para passar uma tabela de argumentos opcionais para o bind.

### **basic_consume**
* **queue(string)** O nome da fila que será consumida.

* **consumer_tag(string)** É um identificador que pode ser passado para identificar o consumidor no canal de conexão. Se for passado em branco o servidor criará um identificador automaticamente.

* **no_local(boolean)** Definido este atributo o servidor não envia mensagens para a conexão que a publicou.

* **nowait(boolean)** Faz com que o servidor não espere o reconhecimento do consumidor para a mensagem. Ele assume que como a mensagem foi entregue o processamento vai ser bem sucedido.

* **exclusive(boolean)** Faz com que o consumidor tem acesso exclusivo a esta fila.

* **nowait(boolean)** Com este atributo ligado o método não vai esperar a resposta do servidor.

* **callback(callback)** A função que será executado para cada mensagem consumida.

* **arguments(AMQPTable)** Serve para passar uma tabela de argumentos opcionais para o consumo.

## Conclusão
Mais uma vez fica aqui o meu muitíssimo obrigado a todos que me acompanharam nessa jornada que serve como pontapé inicial no mundo da mensageria. Demonstrei os primeiros passos e um pouco do podemos fazer usando o  RabbitMQ.

Há muita coisa para ser estudada ainda. Mas com o conhecimento adquirido com estes artigos você vai poder fazer as suas primeiras implementações com um pouco mais de confiança e criar aplicações bastante robustas.

Foi um prazer compartilhar este conhecimento com você e lhe aguardo para os próximos artigos que estão sem uma data definida ainda, mais em breve em chegaram, aguardem. Uma abraço e até a próxima.

## Referências

https://www.rabbitmq.com/amqp-0-9-1-reference.html

https://www.youtube.com/watch?v=FcF5iufd2P0

https://www.youtube.com/watch?v=RnIwwm00UPM
