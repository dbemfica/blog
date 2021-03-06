## Introdução

Hoje quero voltar a falar sobre o Redis, se você não sabe do que se trata o Redis eu tenho um artigo explicando o que é e os primeiros passos que você pode acessar clicando no link [Começando com o Redis](https://diogobemfica.com.br/comecando-com-o-redis/). Mas para resumir o Redis é um banco de dados não relacional armazenando os dados na memória, ele funciona basicamente com uma estrutura de chave valor. Muito bom para ser usado como cache e para controlar as suas sessões.

Nesse artigo vou mostrar como que com poucos passos podemos configurar o PHP para que ao invés de armazenar as suas sessões em arquivos no disco como ele faz por padrão, ele passe a armazenar as suas sessões dentro do Redis.

## Porque usar o Redis

Usando o Redis nós podemos montar um arquitetura para sua aplicação a onde as sessões vão estar centralizadas em um lugar fora da própria aplicação, nós permitindo ter um controle maior e mais facilitado sobre elas.

Fazendo isso podemos escalar a nossa aplicação horizontalmente sem termos que nos preocupar com a sessão do usuário. Além disso fica muito mais fácil para que possamos fazer interoperabilidade entre linguagens e outras tecnologias, exemplo podemos fazer o NodeJs ou Python ter acesso ao usuário logado no sistema.

## Instalando a extensão do Redis no seu PHP

Para que possamos fazer isso precisamos instalar uma extensão do PHP e isso é diferente dependendo do sistema operacional, vou mostrar esse processo usando o Linux e o Windows.

### No Windows

No Windows a melhor forma que eu achei de fazer a instalação é acessando a página da [PECL do Redis](https://pecl.php.net/package/redis). Aqui você encontra todas as versões da extensão, escolha a versão que preferir, não esqueça de baixar a versão em DLL e não em .TGZ. Depois é só escolher a versão para seu PHP.

Depois de baixar pegue o arquivo **php_redis.dll** colocar na sua pasta de extensões dentro a pasta do seu PHP e adicione essa linha no arquivo **php.ini**.
```shel
extension=php_redis.dll
```


### No Linux
O processo no Ubuntu(que é a distribuição que eu estou acostumado)  e derivados basta usar o seu gerenciador de pacotes. 
```shel
sudo apt install php[verão só PHP]-redis
```
 > Credito que para outras distribuições Linux o processo seja semelhante só usando o gerenciador de pacotes de cada distribuição.

## Testando a instalação

Para verificar se instalação foi feita corretamente basta rodar o comando *php -m* no seu terminal ou cmd caso esteja no Windows.
```shel
php -m
```

Rodando esse comando deve aparecer *redis* entre as extensões instaladas do seu PHP.

![Redis-cli](https://diogobemfica.com.br/multimidia/2019_02_20_redis-cli.png)

Ou se preferir pode visualizar isso na sua página do phpinfo.

![Redis-web](https://diogobemfica.com.br/multimidia/2019_02_20_redis-web.png)

## Configurando o PHP

Agora com a extensão instalada é bem simples. Basta acessar o arquivo **php.ini** encontre essas duas linhas.

```shel
session.save_handler = files
session.save_path = "N;/path"
```

E mude as duas para
```shel
session.save_handler = redis
session.save_path = "tcp://[ip do seu servidor]:[porta do seu servidor]?auth=senha_do_redis"
```
 > Por padrão a porta do redis é 6379 e o auth é opcional, isso é só se o seu redis possui senha para acesso.

Pronto agora toda vez que você estiver usando sessões no seu PHP vai saber que elas estão sendo salvas no seu servidor Redis.

## Conclusão
Recentemente eu precisei fazer o PHP guardar as sessões no Redis, e surgiu a ideia de escrever esse artigo para documentar e compartilhar o processo, é um processo simples mas espero possa ajudar a que precisar.