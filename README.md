# Projeto deploy de Servidor Linux
Sexto projeto do curso nanodegree da Udacity de Desenvolvedor Full-stack. O projeto consiste em configurar um servidor remoto hospedado através do serviço da Amazon Lightsail. Deverá ser incluido no mesmo uma aplicação Flask.

## Requisitos e especificações

### 1. Endereço de IP e porta SSH
- Endereço IP: 3.80.159.17;
- Porta SSH utilizada: 2200.

O login é feito através do comando: `ssh grader@54.145.86.10 -p 2200 -i chave_privada`. 

Obs: A chave privada não está disponível neste documento.

### 2. URL da aplicação web
- http://3.80.159.17


### 3. Configurando o servidor
Configuração do servidor:
- Amazon Lightsail;
- Plataforma: Linux/UNIX;
- SO: Ubuntu 16.04 LTS;
- Plano de 512 MB RAM, 1 vCPU, 20 GB SSD.

Configuração do Ubuntu:
- Atualização de pacotes feitas em 23/05/2019;
- Criação de um usuário chamado grader;
- Poderes de sudo foram cedidos ao grader copiando o arquivo `/etc/sudoers.d/ubuntu` e renomeando para grader dentro de seu conteúdo;
- Gerado o novo par de chaves localmente, copiando a chave pública e inserindo em `.ssh/authorized_keys` dentro de `/home/grader/`, com as devidas permissões (700 para `.ssh` e 600 para `.ssh/authorized_keys`);
- Desativação do login via root editando o arquivo `/etc/ssh/sshd_config`, inserindo o trecho `PermitRootLogin no`;
- Edição de `/etc/ssh/sshd_config`, mudando a porta para 2200;
- Permissão de algumas portas no firewall com o `sudo ufw allow`, ativando www, 2200/tcp e ntp. Foi desativada a porta ssh padrão com `sudo ufw deny` e por fim o firewall foi ativado com `sudo ufw enable`;
- Alteração do fuso horário para UTC com `sudo dpkg-reconfigure tzdata`.

Configuração de rede:
- HTTP: 80 - TCP;
- NTP: 123 - UDP;
- SSH: 2200 - TCP.

Pacotes instalados:
- apache2; `sudo apt-get install apache2`
- python-pip; `sudo apt-get install python-pip`
- libapache2-mod-wsgi; `sudo apt-get install python-setuptools libapache2-mod-wsgi`
- git; `sudo apt-get install git`
- postgresql; `sudo apt-get install postgresql`
- psycopg2; `sudo apt-get -qqy install postgresql python-psycopg2`

Configuração do Postgresql:
- Criação de usuário catalog com senha grader e do banco de dados catalog;
	```sh
	postgres=# CREATE DATABASE catalog;
	postgres=# CREATE USER catalog;
	```
    ```sh
	postgres=# ALTER ROLE catalog WITH PASSWORD 'grader';
	```
- Dar ao usuário catalog privilégios de alterar seu conteúdo.

	```sh
	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```

Configuração da aplicação:
- Clonagem do projeto de [Catálogo de Itens](https://github.com/igrdnts/projeto-catalogo) no diretório `/var/www/FlaskApp`;
- Renomear o nome do projeto `sudo mv ./projeto-catalogo ./FlaskApp`;
- Renomear o `application.py`para `__init__.py`;
- Mudar o `create_engine('sqlite:///sportslist.db')` para `create_engine('postgresql://catalog:grader@localhost/catalog')` nos seguintes arquivos:
	- `database_setup.py`
    - `populate_db.py`
    - `__init__.py`
- Instalar o pip `sudo apt-get install python-pip`
- Instalar as dependências com `sudo pip install -r requirements.txt`
- Instalar o psycopg2 com `sudo apt-get -qqy install postgresql python-psycopg2
- Criar o schema do database com `sudo python database_setup.py`
- Popular o banco de dados com `sudo python populate_db.py`

Configurar e habilitar o novo host virtual:
- Criar um arquivo FlaskApp.conf para edição com `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
- Adicionar o seguinte código no arquivo para configuração:

```sh
<VirtualHost *:80>
	ServerName 3.80.159.17
	ServerAdmin igor.rocha@ee.ufcg.edu.br
	WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
	<Directory /var/www/FlaskApp/FlaskApp/>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/FlaskApp/FlaskApp/static
	<Directory /var/www/FlaskApp/FlaskApp/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

- Habilitar o host com `sudo a2ensite FlaskApp`

Criando o arquivo .wsgi
- Crie o arquivo com `sudo nano /var/www/FlaskApp/flaskapp.wsgi
- Insira as seguintes linhas de código no arquivo:

```sh
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'super_secret_key'
```

- Reinicie o apache com `sudo service apache2 restart`