Para muitas pessoas fazer comparações, fazer a adição, a subtração e outras operações com datas com a linguagem PHP uma pequena dor de cabeça. Bom pelo menos para mim era e acredito que muitos outros leitores também seja. Mas vim para mostrar que isso vai ficar no passado com a classe DateTime nativa do PHP.

Bom já comecei dizendo ela é nativa então não precisa ativar nenhuma extensão ou baixar nenhum pacote. basta sair usando-a como no exemplo a baixo.
```php
<?php
$data = new \DateTime();
```
> Rodando esse comando vamos ter um objeto do tipo DateTime com data e hora atual e outras informações.
object(DateTime)#1 (3) { ["date"]=> string(26) "2017-03-09 20:51:47.000000" ["timezone_type"]=> int(3) ["timezone"]=> string(17) "America/Sao_Paulo" }

Mas também podemos passar uma data para o objeto criarmos o nosso objeto.
```php
<?php
$data = new \DateTime('2017-03-09 13:05:29'); // no formato do banco de dados
$data = new \DateTime(''09/03/2017 13:05:29''); // no formato americano
```
> Outros formatos de data que você pode passar para criar o objeto você pode conferir na documentação [Formatos de Data e Hora Suportados](http://php.net/manual/pt_BR/datetime.formats.php)

Mas agora temos esse objeto e ele nós muitas possibilidades. Para começarmos podemos simplesmente retornar a data mas no formato que quisermos usando o método format como tempos no exemplo a baixo.

```php
<?php
$data = new \DateTime();
echo $data->format("d/m/Y"); // 09/03/2017
echo $data->format("d/m/Y H:s:i"); // 09/03/2017 20:29:59
echo $data->format("Y"); // 2017
echo $data->format("M"); // Mar
```
> Outros formatos você pode conferir na documentação.
[date](http://php.net/manual/pt_BR/function.date.php).

Tendo esse objeto facilmente podemos fazer comparações entre eles, como descobrir qual data é maior que outra.
```php
<?php
$data1 = new \DateTime('2017-03-09 13:05:29');
$data2 = new \DateTime('2017-03-08 13:05:29');
if($data1 > $data2){
	echo "Data 1 maior que Data 2";
}else{
	echo "Data 2 maior que Data 1";
}
```
> Neste caso vai aparecer na tela “Data 1 maior que Data 2”.

Outra coisa que conseguimos fazer com facilidade é calcular a diferença entre datas. Isso nós fazemos com a ajuda de outra classe nativa, a “DateInterval”. Essa nova classe vem como retorno do método “diff”. Vamos entender isso melhor com um exemplo

```php
<?php
$data1 = new DateTime('2017-10-11');
$data2 = new DateTime('2017-10-13');
$interval = $data1->diff($data2);
echo $interval->format('%d dias'); // 2 dias
```
> Essa classe tem o seu próprio método *format* e tem precisa seguir essas especificações [DateInterval::format](http://php.net/manual/pt_BR/dateinterval.format.php)

Mas a classe DateInterval além de nos permitir fazer comparações com as datas, ela também nos permite adicionar ou subtrair datas dentro da nossa data original. Usamos o método *add()* para adicionar ou *sub()* para subtrair. Para fazer os calculados passamos a classe DateInterval como parâmetro. A classe DataInterval tem uma especificação para ser passado como uma string para o seu construtor.

Vamos explicar se eu passar “P2D5M” onde primeiro passamos P(Período) depois 2D(dois dias) e 5M(5meses). Ou quando quisermos passar tempo como horas, minutos e segundos. Nesse temos “PT5H” onde precisamos passar o P(Período) obrigatoriamente mas sem nenhuma data e já passamos o T(Tempo) com 5H(cinco horas).

```php
<?php
$data = new \DateTime();
$data->add(new \DateInterval("P2Y"));
echo $data->format('d/m/Y H:i:s'); // 09/03/2019 19:57:04

$data->add(new \DateInterval("PT6H"));
echo $data->format('d/m/Y H:i:s'); // 10/03/2019 01:57:04

```
> Pode olhar as especificações aceitas pelo DateInterval no link [DateInterval::__construct](http://php.net/manual/pt_BR/dateinterval.construct.php)


Ainda tem muita coisa para se ver quando o assunto é datas mas espero que com essas novas dicas eu tenho ajudado a melhorar os seus códigos. Espero que isso seja um ponto pé inicial para você se aventurar, se profissionalizar na sua carreira com o PHP.
