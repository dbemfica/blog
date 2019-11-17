Hoje eu quero dar início ao que eu espero que seja uma mini série de artigos aqui no blog falando sobre mensageria, mais especificamente sobre o RabbitMQ. Está serie serve para compartilhar e documentar um pouco dos meus estudos sobre este assunto.

Neste artigo vou dar os primeiros passos para que você entenda e possa desenvolver as suas próprias aplicações com PHP que se integrarem com o RabbitMQ.

## O que é o RabbitMQ
O RabbitMQ é uma aplicação de código aberto que suporta o protocolo AMQP para receber mensagens organizá-las em filas e as disponibiliza para que outras aplicações possam recebê-las. Chamamos o RabbitMQ de um **Message Broker**(Intermediador de mensagens). Quando trabalhamos com o RabbitMQ a nossa aplicação envia as mensagens para uma Exchange e ele se encarrega de organizar as mensagens nas filas. Neste momento novos jargões apareceram, jargões não comuns para desenvolvedores web mas não se preocupe vamos explicá-los.

### AMQP
O AMQP é um acronimo de Advanced Message Queuing Protocol traduzindo é Protocolo avançado de enfileiramento de mensagens. Ele serve para troca de mensagens entre aplicações independente da linguagem. Fazendo uma analogia podemos dizer que o AMQP é como o HTTP é para aplicações web convencionais.

### Exchange
Uma Exchange é quem vai definir para qual fila as mensagens recebidas vão. Fazendo uma analogia podemos dizer que uma Exchange é como um roteador de internet. No RabbitMQ a sua aplicação somente envia mensagens para uma Exchange, nunca para uma fila diretamente. O RabbitMQ possui quatro tipos de Exchange, são elas **direct**, **fanout**, **headers** e **topic**. Falaremos mais sobre os tipos Exchanges nos próximos artigos onde vamos demonstrar um exemplo de implementação com o PHP para cada uma delas.

### Filas(Queue)
Uma fila basicamente é a onde as mensagens enviadas vão ser armazenadas para serem consumidas por outras aplicações. No RabbitMQ as mensagens não podem ser ordenadas estamos se falando de uma fila, a primeira que entra é a primeira a sair. E também não podemos apagar uma mensagens individualmente, só existe a opção de limpar a fila inteira.

### Mensagem
Uma Message(mensagem) é nada mais que os dados enviados pela sua aplicação. Fazendo uma analogia a mensagem é como uma requisição HTTP.

## Instalando o RabbitMQ
Para fazer a instalação achei melhor mostrar como fazer usando o Docker, isso vai facilitar muito o nosso trabalho e para ambientes de desenvolvimentos e ou estudo um container Docker vai ser mais que o suficiente.

Já que vamos usar o Docker você vai precisar ter o Docker instalado no seu computador, mas tendo ele instalado basta rodar o seguinte comando no seu terminal.
```bach
docker run -d --hostname my-rabbit --name rabbitmq -p 15672:15672 -p 5672:5672 rabbitmq:management-alpinedr
```

O RabbitMQ tem uma interface web para que você possa fazer o seu gerenciamento, então depois de rodar o comando acima você pode acessar o endereço *http://localhost:15672*. Você se deparar com tela semelhante a imagem abaixo.

![Login no RabbitMQ](https://diogobemfica.com.br/multimidia/2019_11_17_login_rabbitmq.png)

Você usa **guest** como usuário e senha para realizar o login. Depois do login realizado você vai se deparar uma tela semelhante a imagem de abaixo.

![Login no RabbitMQ](https://diogobemfica.com.br/multimidia/2019_11_17_overview_rabbitmq.png)

Agora está tudo pronto para nos próximos artigos nós criarmos os nossos primeiros scripts em PHP para publicar e consumir mensagens usando o RabbitMQ.

Para finalizar este artigo acho importante comentar a onde você deve ou não usar o RabbitMQ, porque talvez mais importante que saber onde usar uma determinada tecnologia é justamente saber a onde não usá-la.

## Onde usar o RabbitMQ
Você pode usar o RabbitMQ sempre que você precisar fazer a comunicação entre aplicações. Ele é extremamente rápido e seguro mais na minha opinião existem 2 casos onde ele brilha mais, são eles:

Quando não precisa de uma resposta imediata. No caso requisições assíncronas, Operações como enviar um e-mail, SMS são ótimos exemplos de requisições a onde o usuário só precisa saber que o email ou o SMS vai ser enviado, não precisa saber exatamente quando isso aconteceu.

Outro caso é quando você sabe que para processar a requisição que o usuário solicitou vai demorar. Aí você não quer deixar ele esperando, você pode usar o RabbitMQ pegar a requisição enviar para uma fila para ser processada e para o usuários que assim que for processado ele vai receber uma notificação do sistema.

Vamos tentar trazer para uma situação mais real, vamos imaginar uma situação hipotética  você tem uma aplicação que precisa se comunicar com uma outra aplicação de terceiro por exemplo um gateway de emissão de notas fiscais. Ai todo final de mês você vai precisar enviar para o gateway todas as notas que você precisa emitir. Se você está começo da vida da sua aplicação vocễ deve ter poucos clientes e fazer esse trabalho deve ser algo consideravelmente rápido. Mas há medida que o tempo vai passando que você vai conseguindo mais clientes um processo como esse que demora poucos segundos começa a demorar alguns minutos. E coitado do seu usuarios que vai ter que ficar esperando este processamento terminar. Nós precisamos arranjar um jeito para que o seu usuário não precisa esperar o processo terminar e da mesma forma garantir que o processa seja realizado.

É nessas horas que entra o RabbitMQ, com ele podemos facilmente montar está arquitetura. A sua aplicação envia a os dados(mensagens) que precisam ser processado para ele e ele armazena e vai disponibilizar os dados(mensagens) para a outra aplicação sua que fara trabalho pesado.

## Onde não usar RabbitMQ
Como eu disse antes o RabbitMQ pode ser usado sempre que você quiser fazer comunicação entre aplicações. Mas na minha opinião existem casos em que fazer uma comunicação usando HTTP é melhor que o usar o RabbitMQ. Como nos casos em que você precisa fazer uma comunicação síncrona a onde o seu usuário precisa saber na hora se sua solicitação foi resolvido ou não.

Até existe um Pattern(padrão) chamado RPC(Remote Procedure Call) para fazer comunicações síncronos com AMQP consequentemente com o RabbitMQ mais cabe a você fazer a implementação desse pattern e fazer isso pode acabar sendo mais custoso do simplemente usar o HTTP.
 > Caso tenha curiosidade você pode acessar um link do proprio RabbitMQ falando sobre esse RPC (RabbitMQ tutorial - Remote procedure call [RPC](https://www.rabbitmq.com/tutorials/tutorial-six-python.html)

Claro isso vai de caso a caso, sempre cabe a você fazer uma avaliação e tomar a melhor decisão para o seu cenário.

## Conclusão
Nos próximos artigos vamos detalhar os tipos de Exchanges suportadas pelo RabbitMQ e vamos exemplos de implementação para cada uma dela. A minha ideia é lançar um artigo novo todo o Domingo, um para cada tipo de Exchange e eu espero te ver lá. Até Domingo que vem.

## Referência

https://www.rabbitmq.com/

https://www.amqp.org/about/what

https://www.rabbitmq.com/tutorials/tutorial-six-python.html