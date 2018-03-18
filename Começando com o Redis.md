Voltando a falar de ferramentas/tecnológicas que eu descobri recentemente, tenho mais uma para compartilhar com vocês. Estou falando o banco de dados não relacional o Redis.

Essa informação foi tirada do próprio site do Redis.
> Redis is an open source (BSD licensed), in-memory data structure store, used as a database, cache and message broker. It supports data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs and geospatial indexes with radius queries. Redis has built-in replication, Lua scripting, LRU eviction, transactions and different levels of on-disk persistence, and provides high availability via Redis Sentinel and automatic partitioning with Redis Cluster.

Tradução by Google Tradutor
> Redis é uma fonte aberta (BSD licenciado ), armazenamento de estrutura de dados na memória, usado como banco de dados, cache e corretor de mensagens . Ele suporta estruturas de dados como seqüências de caracteres, hashes , listas , conjuntos , conjuntos classificados com consultas de intervalo , bitmaps , hyperloglogs e índices geoespaciais com consultas de raio.

Como está dizendo no site podemos usar o Redis como um banco de dados não relacional e como um cache. Então usando o Redis podemos fazer as nossas consultas nas nossa aplicações de forma incrivelmente rápida sem usar o nosso banco de dados principal.

Ele é muito usado para listar aquelas 5 ultimas noticias presentes no seu site. ou mostrar o total de alunos cadastrados na sua escola ou plataforma de ensino. Isso para mostrar os exemplos mais simples de uso do Redis.

**Instalando o Redis no Linux**
Para instalar em Ubuntu e derivados basta usamos o famoso apt-get
```bash
sudo apt-get install redis-server
```
Para instalar em Ubuntu e derivados basta usamos o famoso yum install
```bash
yum install redis
```
**Instalando o Redis no Mac**
Para ser sincero não tenho um Mac para testar a instalação, encontrei essas passos para fazer a instalação e me pareceu bem simples de instalar, para quem tem um poder testar para nós coloca nos comentários abaixo.

Basta fazer o download do arquivo tar.gz no link [https://redis.io/download](https://redis.io/download) ir seguir os comandos

```bash
#extrair o arquivo
#cd “até a pasta a onde foi feito o download”
make test
make
sudo mv /src/redis-server /usr/bin/
sudo mv /src/redis-cli /usr/bin/
mkdir ~/.redis
touch ~./redis/redis.conf
redis-server
```

**Instalando o Redis no Windows**
O Redis oficialmente ainda não tem suporte oficial para o Windows, mas o grupo de desenvolvedores “Microsoft Open Tech” tem mantido uma versão compatível com o windows 64bits. Como diz no site oficial do Redis
> The Redis project does not officially support Windows. However, the Microsoft Open Tech group develops and maintains this Windows port targeting Win64

Você pode ver mais sobre o projeto do “Microsoft Open Tech” no [github](https://github.com/MSOpenTech/redis)

**Primeiros passos no Redis**
Mas como funciona o redis, vamos dizer que basicamente ele usa a relação de chave e valor. a onde usamos o comando SET para gravar uma informação e comando GET para pegar a informação presente na chave

***SET***
Nós usando o comenta SET gravar a informação no Redis
```bash
#Exemplo SET “chave” “valor”
SET nome “Diogo”
SET “nome completo” “Diogo Bemfica”
```
> Obs: usar as aspas na chave é opcional quando a chave não contém espaços mas se for o caso ela é obrigatória.

Você pode ler a documentação do comando SET nesse link
[SET - Redis](https://redis.io/commands/set)

***GET***
Como dito anterior nós usamos o GET pegar a informação presente na chave

```bash
#Exemplo GET “chave”
GET nome #imprime na tela “Diogo”
GET “nome completo” #imprime na tela “Diogo Bemfica”
```

Você pode ler a documentação do comando GET nesse link
[GET - Redis](https://redis.io/commands/get)

Esse é o funcionamento básico sobre o Redis, mas com isso já podemos realizarmos muitas coisas, basta usar a criatividade.

**Usando o Redis com o PHP**
Agora vamos por a mão na massa, o Redis é um banco de dados que pode ser usado por praticamente todas as linguagens de programação da atualidade. Mas como a minha linguagem mãe é o PHP, vai ser com ele que vou lhes mostrar um exemplo de como usar. Para isso precisamos baixar a biblioteca “[predis](https://github.com/nrk/predis)” no github, usando o (composer)[https://getcomposer.org/]

```bash
composer require predis/predis
```

Com a biblioteca instalada precisamos passar as configurações.
```php
require_once “vendor/autoload.php”;
$redis = new Predis\Client([
    'scheme' => 'tcp',
    'host'   => '127.0.0.1',
    'port'   => 6379,
]);
```

Agora é fácil pois já temos uma objeto é será o responsável por fazer toda a conexão com o Redis, semelhante como o que fazemos quando usamos o [PDO](http://php.net/manual/pt_BR/book.pdo.php)

***SET com o PHP***
Usando o comando SET para gravar a informação do Redis usando o PHP

```php
$redis->set('nome', 'Diogo);
```

***GET com o PHP***
Usando o comando GET para pegar a informação do Redis usando o PHP

```php
$nome = $redis->get('nome');
echo $nome //imprime na tela “Diogo”
```

Agora você já sabe o que é como instalar Redis e como usar ele basicamente com o nosso querido PHP, tem muita coisa para se fazer com o Redis, eu mesmo estou aprendendo. A medida que eu for descobrindo mais coisas vou passando para você. Espero que esse artigo te instigue e pesquisar e aprender mais sobre esse banco dados/cache.

De uma eu coisa eu posso te garantir como o Redis guarda as informações em memória a velocidade de leitura é incomparavelmente mais rápida que o bancos de dados relacionais que guardam as informações no disco. Com o pouco conhecimento que eu te passei aqui dá para fazer muita coisa de deixar a sua aplicação muito mais rápida.