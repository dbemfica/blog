Algum tempo atrás implementei uma API para a empresa a onde estou trabalhando, fazendo algumas pesquisas para sobre qual método deveria utilizar para fazer a autenticação decidi por usar Json Web Token o famoso JWT e hoje quero contar como implementar, como ele funciona e implementar junto de vocês usando minha linguagem do coração o PHP, sem nenhuma biblioteca.

## JSON Web Token
O JWT é um padrão aberto documentado pelo [RFC 7519](https://tools.ietf.org/html/rfc7519), com ele conseguimos transmitir informações garantindo a sua autenticidade, podendo ser usado autenticação de APIs, sistemas ou em ações mais específicas como recuperar a senha de um usuário.

Um JWT consegue ser compacto e autônomo, com isso quero dizer que devido ao seu pequeno tamanho podemos colocar ele dentro de URLs, como um parâmetro do POST ou dentro de um header do HTTP e ainda assim transmitir de forma rápida. E por ser autônomo todo o conteúdo necessário para realizar a autenticação está presente nele mesmo, sem a necessidades de uma consulta no banco de dados ou algo parecido.

## A estrutura do JWT
Um JWT é divido em três partes separadas por ponto ”.”, um header, um payload e uma signature, vamos falar delas separadamente enquanto implementamos nosso token com PHP.

### HEADER
O header basicamente consiste de dois valores: um é o tipo do token, que nesse caso é JWT, e segundo valor é o algoritmo utilizado de hashing como o HMAC SHA-256 ou RSA, nesse caso HS256.

```php
<?php
$header = [
   'alg' => 'HS256',
   'typ' => 'JWT'
];
$header = json_encode($header);
$header = base64_encode($header);
```
Criamos um vetor chamado de $header com as duas informações necessárias e codificamos ela para um JSON e depois para base64. Tendo um retorno semelhante a este.
> eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9

### PAYLOAD
O payload é o corpo do JWT é aqui que você que está criando o JWT vai colocar todas as informações que deseja armazenar. Não abuse da quantidade de informações e você colocar no seu payload para não tornar o seu token um fardo para sua aplicação.

```php
$payload = [
   'iss' => 'localhost',
   'name' => 'Diogo',
   'email' => 'diogo.fragabemfica@gmail.com'
];
$payload = json_encode($payload);
$payload = base64_encode($payload);
```
Usamos mesmo processo de antes, criamos um vetor chamado dessa vez de $payload, armazenamos as informações que queremos armazenar no token. Codificamos ela para JSON e depois para base64. Tendo um retorno semelhante a este.
> eyJpc3MiOiJsb2NhbGhvc3QiLCJuYW1lIjoiRGlvZ28iLCJlbWFpbCI6ImRpb2dvLmZyYWdhYmVtZmljYUBnbWFpbC5jb20ifQ==

> Obs: O JWT tem palavras reservadas recomendadas para serem colocadas dentro do payload, são elas:
* "iss" O domínio da aplicação geradora do token.
* "sub" É o assunto do token, mas é muito utilizado para guarda o ID do usuário.
* "aud" Define quem pode usar o token.
* "exp" Data para expiração do token.
* "nbf" Define uma data para qual o token não pode ser aceito antes dela.
* "iat" Data de criação do token.
* "jti" O id do token.

Por se tratar de recomendações elas não são obrigatórias mas se não quiser usar nenhuma delas recomendo não às sobrescreva para evitar conflito com as bibliotecas que usarem para implementar o JWT.

### SIGNATURE
O signature é a nossa assinatura, vamos pegar o nosso header e nosso payload e codificar com o algoritmo o escolhido anteriormente mais uma chave da nossa escolha. Neste caso usei "minha-senha" como exemplo, por favor utilize uma senha mais segura no seu projeto.

```php
$signature = hash_hmac('sha256',"$header.$payload",'minha-senha',true);
$signature = base64_encode($signature);
```
Se seguirmos o código a cima vamos ter algo parecido com isso.
> SrcDhO9LLSNYPHl7nAWJnLXdH7QCK4oRLfFMz48zgwk=

Agora é só juntamos tudo para termos o nosso token, lembrando que as três partes são separadas por ponto.

```php
echo "$header.$payload.$signature";
```
> eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJsb2NhbGhvc3QiLCJuYW1lIjoiRGlvZ28iLCJlbWFpbCI6ImRpb2dvLmZyYWdhYmVtZmljYUBnbWFpbC5jb20ifQ==.SrcDhO9LLSNYPHl7nAWJnLXdH7QCK4oRLfFMz48zgwk=

Se tudo ocorrer como o esperado temos um token semelhante com o que está acima e um script PHP parecido com este.
```php
<?php
$header = [
   'alg' => 'HS256',
   'typ' => 'JWT'
];
$header = json_encode($header);
$header = base64_encode($header);

$payload = [
   'iss' => 'localhost',
   'name' => 'Diogo',
   'email' => 'diogo.fragabemfica@gmail.com'
];
$payload = json_encode($payload);
$payload = base64_encode($payload);

$signature = hash_hmac('sha256',"$header.$payload",'minha-senha',true);
$signature = base64_encode($signature);

echo "$header.$payload.$signature";
```

## Testando nosso JWT
Agora podemos testar se o nosso token está funcionando. para isso vamos usar o site [JWT.io](https://jwt.io/). Cole o seu token no campo Encoded como mostra a imagem a baixo.
![JWT.io](https://diogobemfica.com.br/wp-content/uploads/2017/11/jwt.io_-1.png)

Podemos ver todas as informações do token e também que o token está inválido, para resolver isso basta ir até sessão VERFY SIGNATURE e colocar a chave usado na hora da assinatura, no nosso caso a chave é "minha-senha".

![JWT.io](https://diogobemfica.com.br/wp-content/uploads/2017/11/jwt.io2_-2.png)
Agora pode ver que o nosso token está marcado como valido e o seu token está funcionando perfeitamente.

## Validando o Token
Agora temos o nosso token prontinho e pronto para ser usado precisamos saber como fazer para a nossa aplicação PHP consiga validá-lo. Vamos imaginar que vocẽ tem uma página php esperando que você passe o token pela url, pensando nisso nós precisamos criar o seguinte código.
```php
<?php

$token = $_GET['token'];

$part = explode(".",$token);
$header = $part[0];
$payload = $part[1];
$signature = $part[2];
```
Vamos capturar o token usando o $_GET e separá-lo novamente com o explode lembrando que usamos o “.” para unir as partes. Assim podemos ter novamente o $header, $payload e o $signature.

Para fazer a validação precisamos pegar o $header e o $payload é a mesma chave que utilizamos anteriormente e passar pelo mesmo processo de assinatura.
```php
$valid = hash_hmac('sha256',"$header.$payload",'minha-senha',true);
$valid = base64_encode($valid);

if($signature == $valid){
   echo "valid";
}else{
   echo 'invalid';
}
```
Se assinatura gerada for igual a anterior então garantimos que o token não sofreu nenhuma alteração e as informações contidas nele estão íntegras. Porque se qualquer alteração for feita no token, tanto no seu header quanto no payload ou na assinatura a nova assinatura não seria igual, e para qualquer um que queira criar um token assinado precisa da palavra chave.

Com essa validação o usuário agora pode acessar o sistema, ou recuperar a sua senha ou que ele mais desejar e tiver acesso para fazer.

Se tudo estiver como o esperado o seu arquivo para validar o seu token ficará assim.
```php
<?php

$token = $_GET['token'];

$part = explode(".",$token);
$header = $part[0];
$payload = $part[1];
$signature = $part[2];

$valid = hash_hmac('sha256',"$header.$payload",'minha-senha',true);
$valid = base64_encode($valid);

if($signature == $valid){
   echo "valid";
}else{
   echo 'invalid';
}
```

É muito importante destacar que o JWT garante a autenticidade e não confidencialidade das informações presentes nele. Então jamais deve se colocar informações sensíveis como senha de usuário dentro do payload, porque como visto no site [JWT.io](https://jwt.io/) essas informações são de fácil acesso. Use o JWT com sabedoria.

Espero que esse conhecimento tenha te ajudado a entender como é o funcionamento básico do JWT e te Incentive a pesquisar mais sobre assunto.

## Referencias
RFC 7519
[https://tools.ietf.org/html/rfc7519](https://tools.ietf.org/html/rfc7519)

O site do JWT.io
[https://jwt.io/](https://jwt.io/)

Screencast do Fábio Vedovelli
[https://www.youtube.com/watch?v=k3KfK0ZS_FY](https://www.youtube.com/watch?v=k3KfK0ZS_FY)


