Emitir notas fiscais eletronicas(NFe) com o PHP hoje em dia é uma tarefa bem tranquila. Nesse artigo venho explicar todo o processo que envolve no envio das nas notas para a receita federal.

Antes mesmo de tocarmos nos assuntos tecnicos quero fazer uma pequena explicaçao sobre o que são as notas fiscais. As notas nada mais são que arquivos XMLs que contem informações dos produtos vendidos ou serviços prestados com todas as informações tributarias nessarias da receita. Esse arquivo é assinado com um certificado eletronico do emitente e enviado para a receita.

Nesse artigo vamos passar por todas as etapas desse processo. Primeiro a montagem do XML; Segundo Assinatura desse XML; Terceiro o envio para a receita e depois vamos fazer um quarto e ultimo passo que é consultar o nosso envio.

Para essa tarefa vamos usar um framework desenvolvido para esse proposito, estamos falando do 
[nfephp-org/sped-nfe](https://github.com/nfephp-org/sped-nfe) criado por [Roberto L. Machado](https://github.com/robmachado) e mantido por ele e pela comunidade. Com esse framework atuamente podemos emitir NFe e NFC-e nas versão 3.10 e 4.0.

### Montar XML
### Assinar XML
### Enviar Lote
### Concultar Protocolo