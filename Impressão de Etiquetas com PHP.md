## Introdução
Hoje quero compartilhar com vocês uma lib open source que eu mesmo desenvolvi, essa lib foi feita para facilitar na tarefa de impressão de etiquetas. Usando a lib do [mPDF](https://mpdf.github.io/) para fazer a impressao em PDF e usamos [PHPPimaco](https://github.com/PronerInformatica/phppimaco) para abstrair a formatação das etiquetas, fazendo que você só se preocupe com o conteúdo e deixa a lib com o formato e dimensões das etiquetas.

Como as etiquetas existem diversas medidas e formatos a lib é especializada nas etiquetas na marca Pimaco e mesma marca das famosas canetas BIC. Na lib existe uma quantidade de formatos padronizados pela Pimaco já configurados. Provavelmente o código da etiqueta que você precise já está na lib.

Vamos por a mão no código.

## PHP Pimaco
Para fazer a instalação vamos precisar do composer
```bash
composer require proner/phppimaco
```
ou adicione isso ao require do seu composer.json
```bash
"require":{
    "proner/phppimaco": "dev-master"
}
```
Para fazer a nossa primeira impressão vamos usar o exemplo a baixo
```php
<?php

require_once __DIR__ . "/vendor/autoload.php";

use Proner\PhpPimaco\Pimaco;
use Proner\PhpPimaco\Tag;

$pimaco = new Pimaco('6182');

$tag = new Tag();
$tag->p("Etiqueta 1");
$tag->setBorder(0.1);
$pimaco->addTag($tag);

$tag = new Tag();
$tag->p("Etiqueta 2");
$tag->setBorder(0.1);
$pimaco->addTag($tag);

$pimaco->output();
```
A primeira coisa que precisamos e instanciar é a classe *Pimaco* passando o código presente na sua embalagem de etiquetas da Pimaco exemplo *6182*. A classe *Tag* é para cada etiqueta que nós precisarmos.

Com esse exemplo temos o seguinte retorno

![Exemplo 1](https://diogobemfica.com.br/wp-content/uploads/2018/10/pimaco_exemplo_1.png)
> Obs: Por padrão a maioria das etiquetas não possui bordas. Mas foi usado o método *setBorder* da classe *Tag* para melhorar a visualização do exemplo.

Viu como foi fácil, com poucos códigos criamos duas etiquetas e só foi preciso nos preocupar com o conteúdo delas. Todo o layout do documento e da etiqueta está por responsabilidade da classe [PHPPimaco](https://github.com/PronerInformatica/phppimaco).

Agora vamos para uma exemplo mais complexo.
```php
<?php

require_once __DIR__ . "/vendor/autoload.php";

use Proner\PhpPimaco\Pimaco;
use Proner\PhpPimaco\Tag;

$pimaco = new Pimaco('6182');

$tag = new Tag();
$tag->setPadding(3);
$tag->img("https://diogobemfica.com.br/wp-content/uploads/2018/10/logo.png")->setHeight(20)->setAlign('right');
$tag->setBorder(0.1);
$tag->barcode('0001', 'TYPE_CODE_128')->setWidth(2.2)->setMargin(array(0,2,1,0))->br();
$tag->p('0001 - Produto de Teste 1')->setSize(3)->br();
$tag->p('R$: 9,90')->b()->setSize(5);
$pimaco->addTag($tag);

$tag = new Tag();
$tag->setPadding(3);
$tag->img("https://diogobemfica.com.br/wp-content/uploads/2018/10/logo.png")->setHeight(20)->setAlign('right');
$tag->setBorder(0.1);
$tag->barcode('0002', 'TYPE_CODE_128')->setWidth(2.2)->setMargin(array(0,2,1,0))->br();
$tag->p('0002 - Produto de Teste 2')->setSize(3)->br();
$tag->p('R$: 29,90')->b()->setSize(5);
$pimaco->addTag($tag);

$pimaco->output();
```
Agora temos etiquetas mais complexas.
![Exemplo 1](https://diogobemfica.com.br/wp-content/uploads/2018/10/pimaco_exemplo_2.png)
> Obs: Agora imprimimos etiquetas com imagem do logo da empresa mais um código de barras do código do produto.

## Conclusão
Espero que tenha gostado desta dica rápida. E que essa lib possa ajudar vocês na impressão das suas etiquetas.
