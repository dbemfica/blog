## Introdução
Hoje quero compartilhar com vocês uma lib open source que eu mesmo desenvolvi, essa lib foi feita para facilitar a tarefa de impressão de etiquetas. Usando a lib do [mPDF](https://mpdf.github.io/) para fazer a impressao em PDF a [PHPPimaco](https://github.com/PronerInformatica/phppimaco) abstrair a formatação das etiquetas, fazendo que você só se preocupe com o conteúdo e deixa a lib com o formato e dimensões das etiquetas.

Como as etiquetas existem diversas medidas e formatos a lib é especializada nas etiquetas na marca Pimaco e mesma marca das famosas canetas BIC. Possui uma quantidade de formatos padronizados para Pimaco já configurados. Vamos por a mão no código.

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
A primeira coisa que precisamos e extansiar a classe *Pimaco* Passando o código presente na sua embalagem de etiquetas da Pimaco exemplo *6182*. A pos isso extansiamos a classe *Tag* que é cada etiqueta no nosso documento e classe *Pimaco* é o nosso documento.

Com esse exemplo temos o seguinte retorno

![Exemplo 1](https://diogobemfica.com.br/wp-content/uploads/2018/10/pimaco_exemplo_1.png)



Obs: Por padrão a maioria das etiquetas não possui bordas. Mas foi setado uma borda de 0.1 para etiquetas para melhorar a visualização do exemplo.