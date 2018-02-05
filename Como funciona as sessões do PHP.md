Poucas pessoas sabem como funcionam as sessões do php, eu mesmo até pouco tempo atrás não sabia como ele trabalhava. As coisas funcionavam quase como mágica era só dar um *session_start()*. No começo do arquivo PHP e usar o variável global *$_SESSION* que a informação trafegava de página em página dentro da minha aplicação, ela estava lá a qualquer hora ou lugar, só saia quando eu fechava o meu navegador ou depois de um tempo sem mexer com o sistema.

Para começar acho importante falar que quando navegamos usando o navegador usamos o protocolo HTTP e este protocolo é *stateless*(ele não guarda estado) mas o que quer dizer não guarda estado, explicando de uma forma descomplicada o protocolo HTTP trata cada requisição(abertura de página) como unique. Então você acessa um site, requisita os textos e imagens do mesmo e o protocolo faz a transferência dessas informações mas ele não as armazena. Então a cada requisição as informações todas elas são solicitadas de novo.

Mas como as *$_SESSION* do PHP trafegam por todo a minha aplicação se elas são apagadas a cada requisição. Bom o que o PHP faz nada mais é do que uma gambiarra, na verdade muitas linguagens fazem isso mas vamos ao que interessa. Para o PHP fazer isso ele se utiliza de cookies, isso mesmo cookie, a sua sessão nada mais é do que um cookie. 

Mas vamos com calma não estou falando que é da mesma forma que variável global *$_COOKIE*, até mesmo porque se você tentar usar essa variável para pegar as informações de uma variável $_SESSION não vai funcionar. O PHP faz isso da seguinte forma ele criar um arquivo no HD do seu servidor dentro de uma pasta temporária, um arquivo parecido com

sess_0cc175b9c0f1b6a831c399e269772661

É neste arquivo que ficam todas informações presentes na variável global *$_SESSION*. Você pode até ver essas informações dentro do arquivo, indo do no seu php.ini e achando a pasta temporária que ele usa, elas não estão criptografadas e podem ser lidas normalmente.

E ao mesmo tempo ele cria um cookie(por padrão com o nome PHPSESSID) a onde o valor desse cookie é "0cc175b9c0f1b6a831c399e269772661" o mesmo nome do arquivo só que sem o prefixo "sess_" então quando você dá um *session_start()  ele acessa o seu cookie para saber o nome do arquivo e vai nutrir a variável global *$_SESSION*  com os valores presentes nesse arquivo.

Dessa forma ele consegue guardar as informações mesmo trabalhando com inúmeras requisições. Agora você sabe a mágica das variáveis de sessão funciona no PHP.
