Hoje eu gostaria de falar sobre um dos principais aspectos da segurança da informação, estou falando da criptografia. Neste artigo vou mostrar como podemos usar o PHP para trabalhar com os principais algoritmos de criptografia suportados pelo PHP.

Mas antes de irmos aos detalhes sobre como fazer a implementação acho importante comentar que vou cobrir aqui dois tipos de criptografia, um deles é a criptografia do tipo Hash e outro que vou chamar de criptografia tradicional.

# Criptografia do tipo Hash
Uma criptografia hash é aquele que é impossível de se desfazer, então quando texto é criptofado dessa forma não é possível reverter o processo e recuperar a mensagem original. Por isso costuma-se dizer que uma criptografia hash é uma criptografia de mão única, não podendo voltar atrás depois.

Você pode se perguntar porque que nós vamos criptografar uma mensagem de uma forma que não vamos poder recuperar depois. Na verdade isso é muito útil como vamos ver nos exemplos abaixo.

Vamos imaginar um sistema de login com usuário e senha, nesse sistema por questão de segurança é muito importante que não guardemos as senhas dos seus usuários no formato de texto puro, você deve criptografar as senhas de uma forma que nem você consiga descriptografar depois. Isso vai aumentar muito a confiança dos seus usuários no seu sistema. Mas então como eu vou saber se o usuário digitou a senha correta para realizar o login?

Os algoritmos de hash basicamente servem para garantir que a mensagem digitada não tenha sofrido alterações. Já que qualquer alteração na mensagem vai gerar um hash diferente. Então se o seu usuário não digitar exatamente a mesma senha que ele digitou no processo de cadastro e você passar a senha informada para login pelo mesmo algoritmo o hash será diferente do que você armazenou.

Como o hash consegue garantir a integridade da informação ela também pode ser usada para garantir a integridades de programas, como exemplos desses caso temos algumas imagens Linux como o Ubuntu e o Mint que usam o hash *sha256* para fazer a checagem de suas ISO depois do download. Elas te oferecem um hash gerado por esse algoritmo para que você passe a ISO que você recém baixou por ele, se o hash que você obteve for igual a que eles te passaram quer dizer o download aconteceu sem problemas e a ISO está perfeitamente intacta.

Bom agora que já entendemos um pouco sobre a criptografia do tipo hash podemos ver alguns dos algoritmos mais famosos desse tipo de criptografia. Vou falar dos algoritmos *md5*, *sha256* e *bcrypt*.

## md5
O md5 é um hash hexadecimal de 32 caracteres. Este algoritmo foi muito utilizado no começo dos anos 2000 para fazer a criptografia das senhas armazenadas no banco de dados, hoje em dia ele é fortemente não recomendado. Mas o md5 deixou seu lugar na história como um dos hash mas usados em aplicações com PHP.

Para usar o md5 com PHP temos duas opções, podemos usar a função *md5* como podemos ver no exemplo abaixo.
```php
$string = “meu texto a ser encriptado”;
echo md5($string); //8e3d8ef04cecd4c1a879da66285b212a
```
 > Para mais detalhes sobre a função md5 você pode acessar a documentação clicando  [aqui](https://www.php.net/manual/pt_BR/function.md5.php)

E também podemos usar a função *hash* também nativa desde a versão 5.1 do PHP. Para está função precisamos dizer qual algoritmo queremos usar, neste caso o md5. Vejamos o exemplo abaixo.
```php
$string = “meu texto a ser encriptado”;
echo hash('md5', $string) //8e3d8ef04cecd4c1a879da66285b212a
```
 > A função *hash* permite que você use diversos algoritmos de hash bastando mudar o primeiro parâmetro. Para mais detalhes e quais são os algoritmos aceitos pelo função hash basta acessar a documentação clicando  [aqui](https://www.php.net/manual/pt_BR/function.hash.php)


## sha256
O sha256 pertence a família do sha2 que é uma evolução do sha1 e é um algoritmo que gera uma hash hexadecimal de 64 caracteres. Semelhante ao *md5* este algoritmo também sempre vai possuir a mesma saída dada a mesma entrada. Como mencionado anteriormente ele foi muito utilizado pelas distribuições Linux para checar a integridade das ISO baixas pelos usuários.

Para criar uma hash sha256 com PHP basta usar a função nativa *hash* semelhante como fizemos com o md5. Veja o exemplo abaixo.
```php
$string = “meu texto a ser encriptado”;
echo hash('sha256', $string) //7ca2322d6d7aa0adeb942a0be7e7b9d0c56c42dc118e29bf76e6db3188ee04d3
```

## bcrypt
O bcrypt vai sempre retornar uma hash com 60 caracteres. Para que ele possa ser gerado vai ser preciso informar duas coisas. São eles o salt e um custo.

O salt é uma string que você informa no momento de gerar a hash, assim garantindo que somente você ou no caso o seu sistema consiga gerar essa hash novamente já que somente você colocou exatamente a mesma string para gerar o hash. O seu salt precisa ser uma string com 22 caracteres formadas somente por números e letras. No PHP podemos deixar que ele gere um salt de forma aleatória.

Já o custo precisa ser um número inteiro que pode variar entre 4 e 31.

Para gerar uma hash bcrypt com PHP nós usamos a função *password_hash* presente desde a versão 5.5. Vejamos um exemplo.

```php
$string = “meu texto a ser encriptado”;

$options = ['cost' => 8];

$hash = password_hash($string,  PASSWORD_BCRYPT, $options); //a hash será diferente a cada execução
```

Neste caso não passamos o *salt* já que por padrão a função *password_hash* se encarrega de gerar um *salt* diferente para cada execução.

Você até pode informar o seu próprio *salt* como podemos ver no exemplo abaixo.
```php
$string = “meu texto a ser encriptado”;

$options = [
    'cost' => 8,
    'salt' => 'u85YimNH9IbppexoPkz155'
];

$hash = password_hash($string,  PASSWORD_BCRYPT, $options); //agora que você informar o salt o hash será sempre o mesmo
//$2y$08$u85YimNH9IbppexoPkz15ujAP6sPJExGV0DpxhWYPJoOTsZcEsvY6
```

É uma recomendação do próprio PHP que você não informe o salt e deixa para ele gerar um dinamicamente. Isso fará com que hash mude a cada execução e torna a sua aplicação mais segura.

Outra coisa importante que deve ser comentada, nos exemplos foi usado a constante *PASSWORD_BCRYPT* mas existe uma chamada de *PASSWORD_DEFAULT* que atualmente tem o mesmo valor do da *PASSWORD_BCRYPT*. Mas o PHP recomenda que se use a *PASSWORD_DEFAULT* assim quando em novas versões o PHP atualizar para novos algoritmos mais seguros a sua aplicação se atualizará automaticamente.

Neste momento você pode estar se perguntando, se o hash muda a cada chamada da função como vou verificar a senha que o usuário digitou no meu processo de login. Para este caso existe uma função chamada *password_verify* onde você passa a senha informada pelo usuário como primeiro parâmetro e o hash que você tem aguardado no banco de dados como segundo parâmetro. Aí a função retornará se a senha é válida ou não. Vejamos o exemplo.

```php
$string = “meu texto a ser encriptado”;
$hash = '$2y$08$RzfkyBcyXz.SGb8zqM24xOitSUpaPhfYOiN3UzDbCD3YLO74PYl3.'; //vamos imaginar que essa é senha que você tem salva no banco de dados

if (password_verify($string, $hash)) {
    echo 'A senha é válida';
} else {
    echo 'A senha é inválida.';
}
```
 > Para mais detalhes sobre a função password_verify você pode acessar a documentação clicando  [aqui](https://www.php.net/manual/pt_BR/function.password-verify.php)

# Criptografia tradicional
Agora vamos falar do que eu costumo chamar de criptografia tradicional. Nesse tipo de criptografia diferente da tipo hash é possível descriptografar a mensagem e recuperar-la para a mesma forma que ela estava antes do processo de criptografia.

Este tipo de criptografia tem como objetivo de proteger a mensagem para que somente as pessoas(no nosso caso alguma aplicação) possa le-la. Sendo muito usado para enviar dados que possuam a necessidade de serem sigilosas.

Para que conseguimos fazer isso normalmente usamos alguns algoritmos especializados em fazer essa tarefa mais uma chave para fazer a criptografia. Assim somente com essa chave é possível fazer o processo de descriptografar e recuperar a mensagem.

No PHP podemos usar as funções *openssl_encrypt* para criptografar e a *openssl_decrypt* para descriptografar. Elas são mais conhecidas e vamos usar elas paras os nossos exemplos.

```php
$string = "meu texto a ser encriptado";
$algoritmo = "AES-256-CBC";
$chave = "minha_chave";
$iv = "wNYtCnelXfOa6uiJ";

$mensagem = openssl_encrypt($string, $algoritmo, $chave, OPENSSL_RAW_DATA, $iv);
echo base64_encode($mensagem); //codificada em base64 para conseguirmos enviá-la em transtornos
//saida da função é "KvMVCrccROwq0+gq1xAqK1lXvNYsFWv7xGgmEPdyCts="
```
 > Para mais detalhes sobre a função openssl_encrypt você pode acessar a documentação clicando  [aqui](https://www.php.net/manual/pt_BR/function.openssl-encrypt.php)

Vamos explicar cada parâmetro usado no na função
 * **$string** é a mensagem que queremos criptografar
 * **$algoritmo** é o algoritmo usado para gerar a mensagem criptografada. Existem diversos algoritmos suportados pela *openssl_encrypt* e para ver a lista deles acesso o link clicando [aqui](https://www.php.net/manual/pt_BR/function.openssl-get-cipher-methods.php)
 * **OPENSSL_RAW_DATA** (opcional) Esta define o tipo de saída da função, neste caso em bytes. Caso não informe essa define o PHP vai disparar um **PHP Warning**.
 * **$iv** (opcional) É o vetor de inicialização, mas podemos chamar ele de *salt* que usamos nos processos de criptografia com *hash*. Usamos ele para que o resultado da criptografia seja diferente mesmo que *$string* seja a mesma. o **$iv** é um parâmetro de 16 caracteres opcional. Mas o PHP vai disparar um **PHP Warning** caso você não o informa. E caso você use um você precisa dele também para conseguir descriptografar.

Agora temos no nossa mensagem criptografada basta usarmos a função *openssl_decrypt* os parâmetros usados são os mesmos da função *openssl_encrypt*.

 ```php
$mensagem = "KvMVCrccROwq0+gq1xAqK1lXvNYsFWv7xGgmEPdyCts=";
$algoritimo = "AES-256-CBC";
$chave = "minha_chave";
$iv = "wNYtCnelXfOa6uiJ";

$mensagem = openssl_decrypt(base64_decode($mensagem), $algoritimo, $key, OPENSSL_RAW_DATA, $iv);
echo $mensagem; //já que codificamos a mensagem em base64 foi preciso decodificá-la.
//saída da função é "meu texto a ser encriptado"
```
> Para mais detalhes sobre a função openssl_decrypt você pode acessar a documentação clicando  [aqui](https://www.php.net/manual/pt_BR/function.openssl-decrypt.php)

Com essas duas funções conseguimos usar o PHP para enviar mensagem de uma forma segura e garantindo o sigilo delas.

## Conclusão
Neste artigo eu quis mostrar os dois tipos de de criptografia, do tipo hash e do tipo tradicional, mostrar a diferença entre elas e quando usar ou outra. Espero que tenha contribuído para os seus estudo e com isso você conseguirá fazer aplicações mais seguras.

## Referência

https://www.php.net/manual/pt_BR/function.md5.php

https://www.php.net/manual/pt_BR/function.sha1.php

https://www.php.net/manual/pt_BR/function.hash.php

https://www.php.net/manual/pt_BR/function.password-hash.php

https://www.php.net/manual/pt_BR/function.password-verify.php

https://www.php.net/manual/pt_BR/function.openssl-encrypt.php

https://www.php.net/manual/pt_BR/function.openssl-decrypt.php

http://blog.thiagobelem.net/criptografando-senhas-no-php-usando-bcrypt-blowfish

https://tutorials.ubuntu.com/tutorial/tutorial-how-to-verify-ubuntu#0

https://linuxmint.com/verify.php
