# Configuração do servidor para o blog da Smarttbot

## Ambiente
Amazon AWS (nível gratuito)

### Instância
Ubuntu Server 16.04 LTS (HVM), SSD Volume Type (64-bits) - Type: t2.micro
- Permissões de entrada: HTTP / HTTPS

## Domínio
smarttbotblog.tk

## Acesso ao servidor via SSH
```
$ chmod 400 lellis-smarttbot.pem
$ ssh -i "lellis-smarttbot.pem" ubuntu@ec2-18-218-116-202.us-east-2.compute.amazonaws.com
```

## Passos de configuração (SSH)

### Instalação do nginx (v1.12)

```
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:nginx/stable
$ sudo apt-get update
$ sudo apt-get install nginx
$ sudo service nginx start
 ```

### Instalação MariaDB (v10.2)
```
$ sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8
$ sudo vi /etc/apt/sources.list.d/mariadb.list
```

Insira:
```
deb [arch=amd64,i386] http://mirror.jmu.edu/pub/mariadb/repo/10.2/ubuntu xenial main
deb-src http://mirror.jmu.edu/pub/mariadb/repo/10.2/ubuntu xenial main
```

```
$ sudo apt update
$ sudo apt install mariadb-server
```

### Instalação do PostFix
```
$ sudo apt-get update
$ sudo apt install mailutils
```

#### Configuração do PostFix para somente envio de e-mail
- Edite o arquivo `/etc/postfix/main.cf`:
  - Altere `inet_interfaces = all` para `inet_interfaces = loopback-only`
  - Editar `mydestination`: `mydestination = $myhostname, localhost.$mydomain, $mydomain`
  - Adicionar a linha: `smtp_generic_maps = hash:/etc/postfix/generic`
- Editar o arquivo `/etc/postfix/generic`:

```
root@smarttbotblog     no-reply@smarttbotblog.tk
@smarttbotblog         no-reply@smarttbotblog.tk
```

- Execute os comandos:
```
$ sudo postmap /etc/postfix/generic
$ sudo systemctl restart postfix
```

- Em `/etc/passwd`:
  - Alterar `ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash` para `ubuntu:x:1000:1000:SmarttBot Blog:/home/ubuntu:/bin/bash`
  - Alterar `root:x:0:0:root:/root:/bin/bash` para `root:x:0:0:SmarttBot Blog:/root:/bin/bash`

#### Teste de envio
```
$ echo "This is the body of the email" | mail -s "This is the subject line" seu_email
```

**Observação**: O e-mail será enviado para a caixa de SPAM pois não foi configurado o SPF e DKIM.

### Instalação do PHP (v7.1)
```
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:ondrej/php
$ sudo apt-get update
$ sudo apt-get install php7.1-common php7.1-readline php7.1-fpm php7.1-cli php7.1-gd php7.1-mysql php7.1-mcrypt php7.1-curl php7.1-mbstring php7.1-opcache php7.1-json
```

- Edite o arquivo `/etc/php/7.1/fpm/php.ini` e adicione/edite a seguinte configuração: `cgi.fix_pathinfo=0`.

### Configurando o SSL
```
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install python-certbot-nginx
$ sudo certbot --nginx -d smarttbotblog.tk -d www.smarttbotblog.tk
$ sudo certbot certonly --webroot --webroot-path=/var/www/html/blog -d smarttbotblog.tk -d www.smarttbotblog.tk
$ sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
$ sudo vi /etc/nginx/snippets/ssl-smarttbotblog.tk.conf
```

- Edite o arquivo `/etc/nginx/snippets/ssl-smarttbotblog.tk.conf`:
```
ssl_certificate /etc/letsencrypt/live/smarttbotblog.tk/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/smarttbotblog.tk/privkey.pem;
```

- Execute o arquivo `/etc/nginx/snippets/ssl-params.conf` e insira de acordo com o arquivo [ssl-params.conf](ssl-params.conf).

### Instalação do Wordpress
- Configure o banco de dados:
```
$ mysql -uroot -p
  >> CREATE DATABASE wpdb;
  >> GRANT ALL PRIVILEGES ON wpdb.* TO 'wpuser'@'localhost' IDENTIFIED BY 'wpuser_passwd';
  >> FLUSH PRIVILEGES;
```

- Execute os comandos:
```
$ sudo mkdir -p /var/www/html/blog
$ wget -q -O - http://wordpress.org/latest.tar.gz | sudo tar -xzf - --strip 1 -C /var/www/html/blog
```

- Configure as permissões:
```
$ sudo chown www-data: -R /var/www/html/blog
```

- Crie o arquivo `/etc/nginx/sites-available/blog` com os dados em [nginx-blog](nginx-blog);

- Execute os comandos:
```
$ sudo ln -s /etc/nginx/sites-available/blog /etc/nginx/sites-enabled/blog
$ sudo rm /etc/nginx/sites-enabled/default
$ sudo service nginx restart
```

- Acessar `smarttbotblog.tk`
- Insira as configurações do banco de dados e defina o administrador.
