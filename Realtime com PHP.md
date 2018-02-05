Recentemente eu descobri uma ferramenta/plataforma muito boa, essa ferramenta é o Pusher com ela você implementa o Realtime dentro da sua aplicação de forma muito simple e sem dor de cabeça usando várias linguagens.

O Pusher você precisa criar uma conta de forma gratuita e ganha 100 conexões e 200k de mensagens diárias, Isso para quem quer testar e até mesmo pequenas aplicações é o suficiente. Mas ela possui planos pagos para aumentar esses recursos, aí vai da sua vontade e do seu bolso.

Nesse artigo vou mostrar como montar uma pequeno chat usando o Pusher, o PHP e um pouco de Jquery e HTML/CSS.

Para começarmos vamos criar uma página html simples
```html
<!DOCTYPE html>
<html>
<head>
    <title>Pusher Test</title>
</head>
<body>
    <h1>Chat</h1>
</body>
</html>
```
> Uma simples página com a estrutura básica do HTML.

Agora como ninguém é de ferro vou carregar o [Bootstrap](http://getbootstrap.com/) para nós ajudar no layout do nosso site
```html
<link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">
```

Agora com o Bootstrap “instalado” vamos criar o corpo do nosso chat
```html
<div class="container">
        <h1>Chat</h1>
        <div class="row">
            <div class="col-md-12">
                <div id="screen" class="screen"></div>
            </div>
        </div>
        <div class="row">
            <div class="col-md-12">
                <div class="form-group">
                    <input id="nome" type="text" class="form-control" placeholder="Nome">
                </div>
                <div class="form-group">
                    <textarea id="mensagem" class="form-control" rows="5" placeholder="Mensagem"></textarea>
                </div>
                <div class="form-group">
                    <button id="btn" class="btn btn-primary">Enviar</button>
                </div>
            </div>
        </div>
    </div>
```

Com isso terminamos o parte HTML/CSS do nosso chat, agora é a vez do nosso querido companheiro Jquery, Ele vai ser o responsável por enviar as mensagens para o Pusher.
```html
<script
src="https://code.jquery.com/jquery-1.12.4.js"
integrity="sha256-Qw82+bXyGq6MydymqBxNPYTaUXXq7c8v3CwiYwLLNXU="
crossorigin="anonymous"></script>
<script>
$("document").ready(function(){
    $("#btn").click(function(){
        var nome = $("#nome").val();
        var mensagem = $("#mensagem").val();
        $("#mensagem").val('');
        $.ajax({
            url: "post.php",
            method:'post',
            dataType: 'json',
            data: {
                nome: nome,
                mensagem:mensagem
            }
        });
    })
})
</script>
```
> O código acima é responsável por pegar as informações preenchidas nos campos “nome” e “mensagem” e enviá-las via post para uma página PHP e vamos criar ainda.

Já enrolei de mais agora vamos usar o PHP a final de contas o título do post é “Realtime com PHP”. Antes de tudo precisamos pegar a biblioteca do Pusher com o composer, basta executar.

```script
composer require pusher/pusher-php-server
```

Então criamos um arquivo PHP para receber os posts enviados pela página anterior. ela é bem simples e pequena com o código abaixo.
```php
<?php
require_once "vendor/autoload.php";
$app_id = 'YOUR_APP_ID';
$app_key = 'YOUR_APP_KEY';
$app_secret = 'YOUR_APP_SECRET';
$pusher = new Pusher( $app_key, $app_secret, $app_id);
$data['nome'] = $_POST['nome'];
$data['mensagem'] = $_POST['mensagem'];
$pusher->trigger('canal', 'enviar-mensagem', $data);
```

> Obs: Você precisa preencher os campo 'YOUR_APP_ID', 'YOUR_APP_KEY', 'YOUR_APP_SECRET' com os valores respectivos providos pelo Pusher após ter realizado na plataforma deles.

Vamos explicar um pouco o código, o Pusher funciona como um WebSocket com, ele utiliza um canal, então na linha 9 onde temos
```php
$pusher->trigger('canal', 'enviar-mensagem', $data);
```
Nós definimos um canal juntamente com um campo, a onde nesse caso o chamei de ‘canal’ e 'enviar-mensagem', você pode usar qualquer nome para eles mas desde que usa os mesmos nomes na parte que vem a seguir.

Com o nosso PHP pronto estamos chegando no fim do post, nós já estamos enviando as mensagens para o Pusher mas precisamos fazer com o que o nosso front end escute as nossas mensagens. Para isso precisamos colocar o seguinte trecho de código nele, vai ser junto ao nosso Jquery.

```html
Pusher.logToConsole = true;
var pusher = new Pusher('YOUR_APP_ID');
var channel = pusher.subscribe('canal');
channel.bind('enviar-mensagem', function(data) {
	$("#screen").append("<p><b>"+data.nome+"</b>: "+data.mensagem+"</p>");
});
```
> Como eu mencionei anteriormente precisamos respeitar o nome do canal e o campo das mensagens que utilizamos no arquivo PHP.

No final o nosso front end vai ficar assim
```html
<!DOCTYPE html>
<html>
<head>
    <title>Pusher Test</title>
    <link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">
    <style>
        .screen{
            border: 1px solid #ccc;
            min-height: 400px;
            margin-bottom: 20px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Chat</h1>
        <div class="row">
            <div class="col-md-12">
                <div id="screen" class="screen"></div>
            </div>
        </div>
        <div class="row">
            <div class="col-md-12">
                <div class="form-group">
                    <input id="nome" type="text" class="form-control" placeholder="Nome">
                </div>
                <div class="form-group">
                    <textarea id="mensagem" class="form-control" rows="5" placeholder="Mensagem"></textarea>
                </div>
                <div class="form-group">
                    <button id="btn" class="btn btn-primary">Enviar</button>
                </div>
            </div>
        </div>
    </div>
    <script
        src="https://code.jquery.com/jquery-1.12.4.js"
        integrity="sha256-Qw82+bXyGq6MydymqBxNPYTaUXXq7c8v3CwiYwLLNXU="
        crossorigin="anonymous"></script>
    <script src="https://js.pusher.com/4.0/pusher.min.js"></script>
    <script>
        $("document").ready(function(){
            $("#btn").click(function(){
                var nome = $("#nome").val();
                var mensagem = $("#mensagem").val();
                console.log(nome);
                console.log(mensagem);
                $("#mensagem").val('');
                $.ajax({
                    url: "post.php",
                    method:'post',
                    dataType: 'json',
                    data: {
                        nome: nome,
                        mensagem:mensagem
                    }
                });
            })
            Pusher.logToConsole = true;
            var pusher = new Pusher('YOUR_APP_ID');
            var channel = pusher.subscribe('canal');
            channel.bind('enviar-mensagem', function(data) {
                $("#screen").append("<p><b>"+data.nome+"</b>: "+data.mensagem+"</p>");
            });
        })
    </script>
</body>
</html>
```

Seguindo esse poucos passos deu para mostrar que com apenas dois arquivinhos conseguimos montar um singelo mas funcional chat usando o WebSocket, criando uma aplicação Realtime usando o nosso PHP. Espero que isso possa te inspirar a estudar mais sobre o assunto.
Todo os código implementado no chat está no meu Github para consulta.
[Blog-chatPusher](https://github.com/dbemfica/blog-chat-pusher)

