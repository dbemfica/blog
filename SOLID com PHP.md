Hoje quero compartilhar com vocês um pouco dos meus estudos sobre padrões e projeto ou designer pattern. Isso vai ser feito com uma sequência de artigos que vou postar posteriormente, um para cada padrão.

Mas antes mesmo de aborta um padrão especifico acho muito importante falar de SOLID que são 5 principais princípios da POO(programação orientada a objeto) para te ajudar a escrever códigos melhores em qualquer linguagem que tenha o paradigma POO. E como não poderia ser diferente eu vou exemplificar o SOLID com exemplos escritos PHP.

Para começar já vou dizer que SOLID é acrônimo de:

**S**ingle responsibility principle(Princípio da Responsabilidade Única).
**O**pen/closed principle(Principio Aberto/Fechado).
**L**iskov substitution principle(Princípio da Substituição de Liskov).
**I**nterface segregation principle(Princípio da Segregação da Interface).
**D**ependency inversion principle(Princípio da inversão da dependência).

Nesse artigos vamos desmistificar basicamente cada uma das letras do SOLID começando com **S**.

## **S**ingle responsibility principle(Princípio da Responsabilidade Única).
O primeiro princípio do SOLID basicamente diz que uma classe deveria ter exclusivamente uma responsabilidade. Diz que quando temos uma classe que não atenda esse princípio devemos divila em mais classes até que isso ocorra.

Vamos criar um exemplo. Vamos criar uma pequena classe para gerenciar nossos relatórios.

```php
<?php
class Report
{
    public function getTitle()
    {
        return 'Report Title';
    }

    public function getDate()
    {
        return '2018-01-22';
    }

    public function getContents()
    {
        return [
            'title' => $this->getTitle(),
            'date' => $this->getDate(),
        ];
    }

    public function formatJson()
    {
        return json_encode($this->getContents());
    }
}
```
Nesse nosso pequeno exemplo a nossa classe tem duas responsabilidades, 1ª e trazer os dados do relatório. 2ª formatar o relatório para o formato JSON. Então estamos ferindo o princípio da responsabilidade única.

Para resolvermos isso vamos fazer o seguinte.
```php
<?php
class Report
{
    public function getTitle()
    {
        return 'Report Title';
    }

    public function getDate()
    {
        return '2018-01-22';
    }

    public function getContents()
    {
        return [
            'title' => $this->getTitle(),
            'date' => $this->getDate(),
        ];
    }
}

class JsonReportFormatter
{
    public function format(Report $report)
    {
        return json_encode($report->getContents());
    }
}
```

Agora foi criado uma nova classe chamada *JsonReportFormatter* responsavel exclusivamente em formatar o relatório em JSON.

## **O**pen/closed principle (Princípio Aberto/Fechado).
O segundo princípio diz que você deve ser capaz de estender um comportamento de uma classe, sem modificá-lo, a classe está aberto para expansão e fechado para alteração.

Sempre que precisarmos criar um novo recurso devemos criar uma nova classe que implemente esse recurso.

Vamos criar um exemplo. Vamos imaginar a situação onde temos que criar uma pequena classe para registrar os logs da aplicação.

```php
<?php
class Logger
{
    public function writeTxt($message)
    {
        //lógica
    }
}
```
Nesse exemplo criamos um arquivo .txt para armazenar a mensagem do log.

Agora se quisermos criar um para criar o arquivo em CSV precisaríamos modificar a classe para algo parecido com isso.

```php
<?php
class Logger
{
    public function writeTxt($message)
    {
        //lógica
    }

    public function writeCsv($message)
    {
        //lógica
    }
}
```

Foi preciso alterar a classe *Logger*, algo que fere o princípio Princípio Aberto/Fechado. A classe tem que está fechada para alteração.

Vamos refatorar o nosso código.
```php
<?php
class Logger
{
    private $writer;

    public function __construct(Writer $writer)
    {
        $this-writer = $writer;
    }

    public function write($message)
    {
        $this-writer->write($message);
    }
}

interface Writer
{
    public function write($message);
}

class Txt implements Writer
{
    public function write($message)
    {
        //lógica
    }
}

class Csv implements Writer
{
    public function write($message)
    {
        //lógica
    }
}
```
Agora sim, implementamos os mesmos recursos que tínhamos antes, mas agora para implementar um novo recurso como escrever em DOC por exemplo, basta criar uma nova classe *Doc* que implemente a interface *Writer* e pronto, não alteramos nada da classe *Logger*.

## **L**iskov substitution principle(Princípio da Substituição de Liskov).
O terceiro princípio é o mais complicado de entender, ele foi escrito pela cientista da computação Barbara Liskov que resumiu o seu princípio com
 > Se q(x) é uma propriedade demonstrável dos objetos x de tipo T. Então q(y) deve ser verdadeiro para objetos y de tipo S onde S é um subtipo de T.

 Já vi algumas interpretações para o princípio da substituição de Liskov, mas o que eu mais acredito que seja o certo é que as classes derivadas podem ser substituíveis por suas classes base e classes irmãs. Resumindo quando temos uma classe B e classe C que extentende da classe A, deveriamos poder trocar a classe B pelo classe A ou pela classe C dentro do projeto sem quebrar o código.

Vamos para o exemplo.
 ```php
<?php
class Logger
{
    public function writer($message)
    {
        //lógica
    }
}

class DatabaseLogger extends Logger
{
    public function __construct(DataBase $database)
    {
        $this->database = $database;
    }

    public function writer($message)
    {
        //lógica
    }
}
class FileLogger extends Logger
{
    public function __construct(FIleManager $fileManager)
    {
        $this->fileManager = $fileManager;
    }

    public function writer($message)
    {
        //lógica
    }
}
```

Nesse exemplo nós temos a classe *Logger*, *DatabaseLogger* e a *FileLogger* o princípio da substituição de Liskov diz que podemos usar tanto a classe *FileLogger* quanto a *DatabaseLogger* na nossa aplicação deve manter o mesmo comportamento, isso vale para qualquer classe que extende da classe *Logger*.

Como mostra no nesse pequeno exemplo, podemos usar todas as classes que vamos atingir o mesmo resultado.
```php
<?php
$logger = new FileLogger($fileManager);
$logger->write('meu log');

$logger = new DatabaseLogger($database);
$logger->write('meu log');

```
Mas ainda temos um problema, ambas as classes tem dependências para funcionar, nesse caso o indicado é usar um pacote para injeção de dependências.

 > Obs: Para isso precisamos sempre cuidar o que vamos retornar nos nossos métodos para não quebrar o código.

## **I**nterface segregation principle(Princípio da Segregação da Interface).
O quarto princípio diz que muitas interfaces específicas são melhores que uma única interface, para não forçar uma classe a implementar um método que ela não vai usar. Precisamos criar pequenas interfaces mais específicas ao invés de termos uma unica generica.

Vamos exemplificar para entender melhor, vamos criar algumas interfaces e classes de aves para nos ajudar.
```php
<?php
interface Aves
{
    public function andar();
    public function voar();
    public function nadar();
}
class Pato implements Aves
{
    public function voar()
    {
        //lógica
    }

    public function nadar()
    {
        //lógica
    }

    public function andar()
    {
        //lógica
    }
}

class Pinguim implements Aves
{
    public function voar()
    {
        //lógica
    }

    public function nadar()
    {
        //lógica
    }

    public function andar()
    {
        //lógica
    }
}

class Andorinha implements Aves
{
    public function voar()
    {
        //lógica
    }

    public function nadar()
    {
        //lógica
    }

    public function andar()
    {
        //lógica
    }
}
```
Nessa estrutura estamos forçando algumas aves a implementar alguns métodos que elas não deveriam possuir. Isso porque existem aves que não voam e aves que não nadam. Como o Pinguim que implementou o método voar já que pinguins não voam. E o mesmo para a Andorinha que não nada.

Vamos ver como criar uma estrutura que siga o princípio da segregação da interface.
```php
<?php
interface Aves
{
    public function andar();
}

interface AvesQueVoam extends Aves
{
    public function voar();
}

interface AvesQueNadam extends Aves
{
    public function nadar();
}

class Pato implements AvesQueVoam, AvesQueNadam
{
    public function voar()
    {
        //lógica
    }

    public function nadar()
    {
        //lógica
    }

    public function andar()
    {
        //lógica
    }
}

class Pinguim implements AvesQueNadam
{
    public function nadar()
    {
        //lógica
    }

    public function andar()
    {
        //lógica
    }
}

class Andorinha implements AvesQueVoam
{
    public function andar()
    {
        //lógica
    }

    public function voar()
    {
        //lógica
    }
}
```
Agora sim, temos as interfaces *AvesQueVoam*, *AvesQueNadam* ao invés de somente uma interface *Aves*.

## **D**ependency inversion principle (Princípio da inversão da dependência).
O último e quinto princípio diz para que uma classe dependa de uma abstração e não de uma implementação.

Vamos exemplificar.
```php
<?php
class Email
{
    public function enviar($mensagem)
    {
        //lógica
    }
}

class Notificacao
{
    public __construct()
    {
        $this->mensagem = new Email;
    }

    public function enviar($mensagem)
    {
        $this->mensagem->enviar($mensagem)
    }
}
```
Neste exemplo temos o que chamamos de acoplamento e uma dependência da classe *Notificacao* cria uma instancia da classe *Email* dentro dela. E o metodo *enviar* faz a utilização da classe *Email* para enviar a notificação por e-mail. Isso fere o princípio da inversão da dependência, porque não desenvolvemos a classe *Notificacao* para uma abstração e sim para uma implementação já que classe *Email* implementa a lógica para o envio do e-mail.

Vamos resolver isso.
```php
<?php
interface MensagemInterface
{
    public function enviar($mensagem);
}
class Email implements MensagemInterface
{
    public function enviar($mensagem)
    {
        //lógica
    }
}

class Notificacao
{
    public __construct(MensagemInterface $mensagem)
    {
        $this->mensagem = $mensagem;
    }

    public function enviar($mensagem)
    {
        $this->mensagem->enviar($mensagem)
    }
}
```

Agora desaclopamos a classe *Email* da classe *Notificacao* estamos trabalhando com a abstração *MensagemInterface*, para *Notificacao* não importa qual classe você está usando e sim que ela implemente a interface *MensagemInterface* porque sabemos que ela vai ter o metodo *enviar* que precisamos. Isso também permite que a classe *Notificacao* use outras classes que implementem a interface *MensagemInterface*.

## Conclusão
Com isso esclarecemos as noções básica para começar a implementar os princípios do SOLID na sua aplicação. Num mundo real nem sempre conseguimos usar todos os princípios em todas as classes do nosso projeto, eles mesmos são como guias para te ajudar a escrever um código mais maduro. Espero ter ajudado a dar os seus primeiros passos para o aperfeiçoamento dos seus códigos.

 > Obs: Todas os códigos que te mostrei aqui são apenas conceituais e servem para nos ajudar a entender o conteúdo abordado nesse artigo.

## Referencias.
<https://www.schoolofnet.com/curso-solid-com-php/>
<https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)>
<http://www.eduardopires.net.br/2013/04/orientacao-a-objeto-solid/>
