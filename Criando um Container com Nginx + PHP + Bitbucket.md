Demorou um pouco, até mais do que devia mas acabei me rendendo ao Docker, o queridinho do mundo DevOps, realmente depois de estudar um pouco sobre ele e criar o meu primeiro container deu para ver que realmente ele veio para ficar e trabalhar com containers realmente é um enorme avanço na questão de virtualização.

No meu trabalho atual eu estou tentando implementar a arquitetura de micro serviços para o nosso enorme monolito. E aí que entra o docker, ele faz com todo essa  implementação e ter vários serviços pequenos trabalhando juntos de forma muito tranquila.

Nesse artigo gostaria de compartilhar a minha experiência que foi criar o meu primeiro container, um container que vai servir uma pequena API que usamos para o aplicativo mobile de vendas da nosso ERP, usando o Nginx + PHP FPM e baixando o projeto de um repositório git privado hospedado no Bitbucket.

A forma de utilizei para criar o container foi usando o Dockerfile, um pequeno arquivo com instruções para criar a imagem do container, onde dentro dessa arquivo coloquei todos os comandos necessários para que o container já saia rodando com um comando após a criação da imagem.

**Criando Dockerfile**

Para começar usamos o comando “FROM” e depois a imagem da onde nós vamos nos basear para criar a nossa imagem, neste caso o ubuntu 16.04.
```bash
FROM ubuntu:16.04
```

Depois precisamos declarar o mantenedor dessa imagem vamos criar.
```bash
MAINTAINER <Seu nome> "<seu e-mail>"
```

Com o comando “ENV” definimos uma variável para o nosso Nginx
```bash
ENV NGINX_VERSION 1.9.3-1~jessie
```

Com o comando “RUN” rodamos comandos dentro do nosso futuro container
```bash
RUN apt-get update && apt-get install -y nginx php7.0-fpm curl git && apt-get clean
```
> É importante o parâmetro -y no apt-get para que ele não te pergunta nada durante a instalação dos pacotes.

Agora definimos as configurações de log e cache do nosso Nginx
```bash
RUN ln -sf /dev/stdout /var/log/nginx/access.log
RUN ln -sf /dev/stderr /var/log/nginx/error.log
VOLUME ["/var/cache/nginx"]
```

Vamos apagar o vhosts padrão que vem na instalação no Nginx e usar o comando “ADD” para adicionar o nosso arquivo com o host.
```bash
RUN rm /etc/nginx/sites-available/default
ADD ./default /etc/nginx/sites-available/default
```
> O arquivo default utilizado nesse artigo está mais abaixo

Agora esse comando é opcional, é só para quem quer vai usar o composer para fazer o build da sua aplicação.
```bash
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && php composer-setup.php && rm composer-setup.php && mv composer.phar /usr/local/bin/composer && chmod a+x /usr/local/bin/composer
ENV COMPOSER_ALLOW_SUPERUSER 1
```
> Este comando “ENV COMPOSER_ALLOW_SUPERUSER 1” vai fazer com que o composer não apresente warning por estar rodando como usuário root.

Vamos nos preparar para baixar a nossa aplicação do Bitbucket, vamos definir o diretório para o git clone com o comando "WORKDIR".
```bash
WORKDIR /var/www/html
```

Para fazermos o git clone vamos usar o comando.
```bash
RUN git clone https://<seu usuario>:<sua senha>@bitbucket.org/<usuario dono do repositório>/<nome do repositório> <nome da aplicação>
```

Estamos quase acabando, agora precisamos expor as portas para que o nosso host tenha acesso às portas do container.
```bash
EXPOSE 80 443
```

Para finalizar e a onde muito gente se atrapalhar ao definir um Dockerfile, apesar de termos feito tudo certo até agora, os nossos serviços do Nginx e PHP FPM estão instalados mas ele não vão iniciar de forma automática, mas para definirmos isso usamos a comando “CMD” para rodar os seguintes comandos.
```bash
CMD service php7.0-fpm start && nginx -g "daemon off;"
```

O seu Dockerfile deve ficar parecido com isso.
```bash
FROM ubuntu:16.04

# Maintainer
MAINTAINER <seu nome> "<seu e-mail>"

ENV NGINX_VERSION 1.9.3-1~jessie

RUN apt-get update && apt-get install -y nginx php7.0-fpm curl git && apt-get clean

# NGINX
RUN ln -sf /dev/stdout /var/log/nginx/access.log
RUN ln -sf /dev/stderr /var/log/nginx/error.log
VOLUME ["/var/cache/nginx"]
RUN rm /etc/nginx/sites-available/default
ADD ./default /etc/nginx/sites-available/default

# COMPOSER
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && php composer-setup.php && rm composer-setup.php && mv composer.phar /usr/local/bin/composer && chmod a+x /usr/local/bin/composer
ENV COMPOSER_ALLOW_SUPERUSER 1

# BUILD
WORKDIR /var/www/html
RUN git clone https://<seu usuario>:<sua senha>@bitbucket.org/<usuario dono do repositório>/<nome do repositório> <nome da aplicação>

EXPOSE 80 443
CMD service php7.0-fpm start && nginx -g "daemon off;"
```

**Arquivo  default**
O arquivo default utilizado na criação desse Dockerfile está logo abaixo, ele precisar estar no mesmo diretório a onde está o Dockerfile para que o comando “ADD” descrito acima funcione.
```bash
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html/<nome da aplicação>;
    index index.php index.html index.htm index.nginx-debian.html;

    server_name _;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.0-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}

```
> Neste arquivo já está configurado as URLs amigáveis.


**Finalizando**
Com esses dois arquivos estamos pronto para criar o nosso container. Vamos dar o nosso primeiro Build(a construção da imagem).
```bash
docker build . -t <nome da imagem>
```

Com a nossa imagem criada vamos levantar o nosso container.
```bash
docker run -d -p 8080:80 <nome da imagem>
```
> Usamos o parâmetro -d para que container não ocupe o terminal e execute em background e o parâmetro -p definimos as portas, onde 8080 é a porta do host e o 80 é a porta do container.

Executando o comando “docker ps” você pode ver todos os container rodando atualmente na sua máquina e perceber que o nosso container está lá.
```bash
docker ps
```

Espero que tenha gostado desse artigo e acho importante avisar que estou começando a brincar com Docker, então tenho pouca experiência. Mas apesar de ser pouca acho que essa informação pode ser útil para quem está com dificuldades para criar os seus primeiros containers.
