### Introdução
Emitir notas fiscais eletronicas(NFe) com o PHP hoje em dia é uma tarefa bem tranquila. Nesse artigo eu tenho o objetivo de explicar todo o processo para quem nunca precisou enviar uma nota siquer, vou dar uma breve explicação do que são e como funciona esse processo de emitir NFe.

### O que são
As notas fiscais eletronicas nada mais são que arquivos XMLs que contem informações dos produtos vendidos ou serviços prestados com todas as informações tributarias nessarias exigidas pela receita. Esse arquivo é assinado com um certificado digital do emitente e enviado para a receita.

### Como vamos fazer isso
Nesse artigo vamos passar por todas as etapas desse processo. Primeiro a montagem do XML; Segundo a Assinatura do mesmo; Terceiro o envio para a receita e depois vamos fazer um quarto e ultimo passo que é consultar o nosso envio para ver se tudo deu certo.

### Mão na massa
Para essa tarefa vamos usar um framework desenvolvido para esse proposito, estamos falando do 
[nfephp-org/sped-nfe](https://github.com/nfephp-org/sped-nfe) criado por [Roberto L. Machado](https://github.com/robmachado) e mantido por ele e pela comunidade. Com esse framework atuamente podemos emitir NFe e NFC-e nas versão 3.10 e 4.0.

## Instalação do nfephp-org/sped-nfe
A instalação do framework é feita atraves do composer com o comando

```bash
composer require nfephp-org/sped-nfe:dev-master
```
 > Se você nunca usou ou não sabe o que é o composer eu sugiro o vídeo do youtube para saber do que eu estou falando.

Com o framework devidamente instalando podemos partir para a montagem do XML

### Montar XML
```php
$nfe = new Make();
$std = new stdClass();
$std->versao = '3.10';
$std->pk_nItem = null;
$elem = $nfe->taginfNFe($std);
$xml = $nfe->getXML();
```
 > Para saber todos os campos suportados pelo framework acesse o link da documentação https://github.com/nfephp-org/sped-nfe/blob/master/docs/Make.md

### Assinar XML
Agora que já temos o nosso XML montato para para o segundo passo que é a assintatura do mesmo.
```php
use NFePHP\NFe\Tools;
use NFePHP\Common\Certificate;

try {
    $tools = new Tools($configJson, Certificate::readPfx($content, $password));
    $response = $tools->signNFe($xml);
   
} catch (\Exception $e) {
    //aqui você trata possiveis exceptions
    echo $e->getMessage();
}
```
 > Mais uma vez caso precise de mais detalhes acesse o link da documentação https://github.com/nfephp-org/sped-nfe/blob/master/docs/metodos/SignNFe.md


### Enviar Lote
```php
use NFePHP\NFe\Convert;
use NFePHP\NFe\Tools;
use NFePHP\Common\Certificate;
use NFePHP\NFe\Common\Standardize;

try {
    //$content = conteúdo do certificado PFX
    $tools = new Tools($configJson, Certificate::readPfx($content, 'senha'));
    $idLote = str_pad($nfemit->id, 15, '0', STR_PAD_LEFT);
    //envia o xml para pedir autorização ao SEFAZ
    $resp = $this->tools->sefazEnviaLote([$xml], $idLote);
    //transforma o xml de retorno em um stdClass
    $st = new Standardize();
    $std = $st->toStd($resp);
    if ($std->cStat != 103) {
        //erro registrar e voltar
        return "[$std->cStat] $std->xMotivo";
    }
    $recibo = $std->infRec->nRec;
    //esse recibo deve ser guardado para a proxima operação que é a consulta do recibo
    header('Content-type: text/xml; charset=UTF-8');
    echo $resp;
} catch (\Exception $e) {
    echo str_replace("\n", "<br/>", $e->getMessage());
}
```
 > Mais uma vez caso precise de mais detalhes acesse o link da documentação https://github.com/nfephp-org/sped-nfe/blob/master/docs/metodos/EnviaLote.md

### Concultar Protocolo
Na ultima etapa temos que consultar para ver se anota foi autorizada ou regeitada.
```php
use NFePHP\NFe\Convert;
use NFePHP\NFe\Tools;
use NFePHP\Common\Certificate;
use NFePHP\NFe\Common\Standardize;

try {
    //$content = conteúdo do certificado PFX
    $tools = new Tools($configJson, Certificate::readPfx($content, 'senha'));
    $idLote = str_pad($nfemit->id, 15, '0', STR_PAD_LEFT);
    //envia o xml para pedir autorização ao SEFAZ
    $resp = $this->tools->sefazConsultaRecibo($recibo);
    //transforma o xml de retorno em um stdClass
    $st = new Standardize();
    $std = $st->toStd($resp);
    if ($std->cStat != 103) {
        //erro registrar e voltar
        return "[$std->cStat] $std->xMotivo";
    }
    $recibo = $std->infRec->nRec;
    //esse recibo deve ser guardado para a proxima operação que é a consulta do recibo
    header('Content-type: text/xml; charset=UTF-8');
    echo $resp;
} catch (\Exception $e) {
    echo str_replace("\n", "<br/>", $e->getMessage());
}
```
 > Mais uma vez caso precise de mais detalhes acesse o link da documentação https://github.com/nfephp-org/sped-nfe/blob/master/docs/metodos/ConsultaRecibo.md