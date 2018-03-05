
Cen�rio
==============================================================================

Zabbix Server\
Zabbix Agent

Debian 9 - Zabbix-Server-MySQL 3.0
==============================================================================

## Instala��o

1- Atualizar sistema

	apt-get update

2- instalar ambiente de LAMP, pr�-requisito para funcionamento do zabbix.

		apt-get install -y apache2
		apt-get install -y mysql-server
		apt-get install -y php7.0
		apt-get install -y php7.0-mysql


3- Instala��o zabbix:

		apt install -y zabbix-server-mysql zabbix-frontend-php zabbix-agent

4- criar base dados chamado zabbix e usu�rio zabbix.

	mysql -u root -p

		> create database zabbix character set utf8 collate utf8_bin;
		> grant all privileges on zabbix.* to zabbix@localhost identified by 'SENHA-USUARIO-ZABBIX';
		> quit;

5- Importar esquemas de tabelas para o "Banco".

	zcat /usr/share/zabbix-server-mysql/schema.sql.gz | mysql -uzabbix -p zabbix
	zcat /usr/share/zabbix-server-mysql/images.sql.gz | mysql -uzabbix -p zabbix
	zcat /usr/share/zabbix-server-mysql/data.sql.gz | mysql -uzabbix -p zabbix

6- Editar arquivo "/etc/zabbix/zabbix_server.conf" procurando as devidas 
linhas exibidas abaixo

	vi /etc/zabbix/zabbix_server.conf

		DBHost=localhost
		#...
		DBName=zabbix
		#...
		DBUser=zabbix
		#...
		DBPassword=SENHA-USUARIO-ZABBIX
		#...

7- Criar a configura��o do "Zabbix" no "Apache2"

	vi /etc/apache2/conf-enabled/zabbix.conf

		## Zabbix
 
		<IfModule mod_alias.c>
		    Alias /zabbix /usr/share/zabbix
		</IfModule>
 
		<Directory "/usr/share/zabbix">
		    Options FollowSymLinks
		    AllowOverride None
 
		    <IfModule mod_php7.c>
		        php_value max_execution_time 300
		        php_value memory_limit 128M
		        php_value post_max_size 16M
		        php_value upload_max_filesize 2M
		        php_value max_input_time 300
		        php_value always_populate_raw_post_data -1
		        php_value date.timezone America/Sao_Paulo
		    </IfModule>
		</Directory>
 
		<Directory ~ "^/usr/share/zabbix/(conf|app|include|local)/">
		    <files *.php>
		    </files>
		</Directory>	

8- Reiniciar "Apache2"

	/etc/init.d/apache2 restart

9- Habiliar e inicializar o "Zabbix Server" e o "Agente"

	systemctl enable zabbix-server
	systemctl enable zabbix-agent
	/etc/init.d/zabbix-server restart
	/etc/init.d/zabbix-agent restart

10- Criar arquivo que receber� as configura��es web.s

	touch /etc/zabbix/zabbix.conf.php
	chown www-data /etc/zabbix/zabbix.conf.php

11- Verifcar as portas abertas no servidor

	netstat -ntpl

## Acessando Front-End

1- Acessar p�gina fdo "Zabbix" em um browser para finalizar os procedimentos de 
configura��o de instala��o

	URL:http://192.168.56.101/zabbix

2- Digitar as informa��es necess�rias nos procedimentos. Informe a senha do banco
"MySQL" quando solicitado.

3- Na p�gina de login do zabbix forne�a as seguintes credenciais

	login: Admin
	senha: zabbix

## Adcionando e configurando Host

1- Acessar "Configura��e>Host" e clicar em "Adicionar Host"

2- Preecha os campos de cria��o da aba "Host".

		> Nome: zabbix_agent_1
		> Grupos: Linux servers;
		> Interface do Agente
			- Endere�o IP: 10.10.10.11
			- Porta: 10050

	#Obs.: "Nome" deve ser o mesmo configurado no arquivo 
               "/etc/zabbix/zabbix_agentd.conf"


Debian 9 - Zabbix-Agent 3.0
============================================================================

1- Instalar zabbix-agent

	apt-get -y install zabbix-agent

2- Editar arquivo de configura��o do zabbix "/etc/zabbix/zabbix_agentd.conf"

		vi /etc/zabbix/zabbix_agentd.conf

			Server=<IP-DO-SERVIDOR>
			#...
			ServerActive=<IP-DO-SERVIDOR>
			#...
			Hostname=zabbix_agent_1
			#...

	#OBS.: "Hostname" deve ser o mesmo que o cadastrado no frotend.

3- Reiniciar zabbix-agent

	service zabbix-agent restart

4- Log do zabbix Agent:

	cat /var/log/zabbix/zabbix_agentd.log
