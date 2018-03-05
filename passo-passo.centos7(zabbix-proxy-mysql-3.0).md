
Cenário
==============================================================================

Zabbix Server\
Zabbix Agent

CentOS 7 - Zabbix-Server-MySQL-3.0
==============================================================================

## Instalação

### 1- Atualizar sistema
	
	yum update

### 2- instalar ambiente de LAMP, pré-requisito para funcionamento do zabbix.

#### 2.2- CentOS 7

		yum install -y httpd
		yum install -y mariadb-server
		yum install -y php
		yum install -y php-mysql

	# Parando e Desabilitando firewall

		systemctl stop firewalld
		systemctl disable firewalld

	# Inciando e habilitando serviços instalados

		systemctl enable httpd
		systemctl start httpd
		systemctl enable mariadb
		systemctl start mariadb

	# configurar senha do mysql
	
		mysql_secure_installation

### 3- Instalação zabbix:

	# Instalção do pacote de configuração do repositório. Este pacote contém arquivos de 
	# configuração de yum

			rpm -ivh http://repo.zabbix.com/zabbix/3.0/rhel/7/x86_64/zabbix-release-3.0-1.el7.noarch.rpm
	
	# Instalação do zabbix

			yum install -y zabbix-server-mysql zabbix-web-mysql

### 4- criar base dados chamado zabbix e usuário zabbix.

	mysql -u root -p

		> create database zabbix character set utf8 collate utf8_bin;
		> grant all privileges on zabbix.* to zabbix@localhost identified by 'SENHA-USUARIO-ZABBIX';
		> quit;

### 5- Importar esquemas de tabelas para o "Banco".

		zcat /usr/share/doc/zabbix-server-mysql/create.sql.gz | mysql -uzabbix -p zabbix

	# OBS.: Informe a senha do usario zabbix criado no "MariaDB"

### 6- Editar arquivo "/etc/zabbix/zabbix_server.conf" procurando as devidas 
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

### 7- Criar a configuração do "Zabbix" no "Apache2"

	vi /etc/httpd/conf-enabled/zabbix.conf

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

### 8- Reiniciar "Apache2"

	systemctl restart httpd

### 9- Habiliar e inicializar o "Zabbix Server" e o "Agente"

	systemctl enable zabbix-server
	systemctl enable zabbix-agent
	systemctl start zabbix-server
	systemctl start zabbix-agent

##### 11- Criar arquivo que receberá as configurações web.s

	touch /etc/zabbix/zabbix.conf.php
	chown www-data /etc/zabbix/zabbix.conf.php

#### 12- Verifcar as portas abertas no servidor

	netstat -ntpl

### Acessando Front-End

#### 1- Acessar página fdo "Zabbix" em um browser para finalizar os procedimentos de 
configuração de instalação

	URL:http://192.168.56.101/zabbix

#### 2- Digitar as informações necessárias nos procedimentos. Informe a senha do banco
"MySQL" quando solicitado.

#### 3- Na página de login do zabbix forneça as seguintes credenciais

	login: Admin
	senha: zabbix

## Adcionando e configurando Host

### 1- Acessar "Configuraçõe>Host" e clicar em "Adicionar Host"

### 2- Preecha os campos de criação da aba "Host".

		> Nome: zabbix_agent_1
		> Grupos: Linux servers;
		> Interface do Agente
			- Endereço IP: 10.10.10.11
			- Porta: 10050

	#Obs.: "Nome" deve ser o mesmo configurado no arquivo 
               "/etc/zabbix/zabbix_agentd.conf"

## Configurando Zabbix proxy

### 1- Acessar "Administração>Proxy" e clicar em Adicionar Host"

### 2- Preencha os campos de criação do host na aba "Proxy" e depois clique em 
"Adicionar".

	> Nome do Proxy: zabbix_proxy
	> Modo do Proxy: Ativo
	> Hosts do Proxy: zabbix-agent-1

### 2.1- Obs.: "Nome do Proxy" deve ser o mesmo configurado no arquivo 
"/etc/zabbix/zabbix_proxy.conf".

### 3- Adicone o Zabbix-Proxy entre os Host à serem monitorados. Acesse 
"Configuraçõe>Host" e clique em "Adicionar Host"

### 4- Preecha os campos de criação da aba "Host".

	> Nome: zabbix_agent_1
	> Grupos: Linux servers;
	> Interface do Agente
		- Endereço IP: <10.10.10.4>
		- Porta: 10050

### 4.1- Obs.: "Nome" deve ser o mesmo configurado no arquivo 
"/etc/zabbix/zabbix_agentd.conf"

##################### Debian 9 Zabbix Proxy 3.0 ########################

## Instalação

### 1- Atualizar sistema

	yum update

### 2- Instalar zabbix apartir do repositório Debian:

	# Instalção do pacote de configuração do repositório. Este pacote contém arquivos de 
	# configuração de yum

		rpm -ivh http://repo.zabbix.com/zabbix/3.0/rhel/7/x86_64/zabbix-release-3.0-1.el7.noarch.rpm

	# 

		yum install -y zabbix-proxy-mysql zabbix-agent

### 3- criar base de dados chamado zabbix e usuário zabbix no "MariaDB"

	mysql -uroot -p

		> create database zabbix character set utf8 collate utf8_bin;
		> grant all privileges on zabbix.* to zabbix@localhost identified by 'SENHA-USUARIO-ZABBIX';
		> quit;

### 4- Importar esquemas de tabelas para o "Mysql"

	zcat /usr/share/zabbix-proxy-mysql/schema.sql.gz | mysql -uzabbix -p zabbix

### 5- Editar arquivo "/etc/zabbix/zabbix_proxy.conf" procurando as devidas linhas 
exibidas abaixo.

		vi /etc/zabbix/zabbix_proxy.conf
	
			#...
			Server=<IP-DO-SERVIDOR>
			#...
			Hostname=zabbix_proxy
			#...
			DBHost=localhost
			#...
			DBName=zabbix
			#...
			DBUser=zabbix
			#...
			DBPassword=zabbix
			#...

	# OBS.: "Hostname" deve ser o mesmo que o cadastrado no frotend.

### 6- Editar arquivo "/etc/zabbix/zabbix_agentd.conf" procurando as devidas linhas 
exibidas abaixo 

		vi /etc/zabbix/zabbix_agentd.conf

			#...
			Server=<IP-do-Zabbix-Proxy>
			#...
			Hostname=zabbix_proxy

	# OBS.: "Hostname" deve ser o mesmo que o cadastrado no frotend.

### 7- Reiniciar zabbix proxy

	/etc/init.d/zabbix-proxy restart

Debian 9 Zabbix-Agent-3.0
============================================================================

### 1- Instalar zabbix-agent

	yum install zabbix-agent

### 2- Editar arquivo de configuração do zabbix "/etc/zabbix/zabbix_agentd.conf"

		vi /etc/zabbix/zabbix_agentd.conf

			Server=<IP-DO-SERVIDOR>
			#...
			ServerActive=<IP-DO-SERVIDOR>
			#...
			Hostname=zabbix_agent_1
			#...

	#OBS.: "Hostname" deve ser o mesmo que o cadastrado no frotend.

### 3- Reiniciar zabbix-agent

	systemctl restart zabbix-agent

### 4- Log do zabbix Agent:

	cat /var/log/zabbix/zabbix_agentd.log
