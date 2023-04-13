# Instalação do Thingsboard Edge no Rapsberry OS

## 1-Em primeiro lugar é necessário atualizar o rpi:
```
sudo apt-get update
sudo apt-get upgrade
sudo reboot
```

## 2-Instalar o java
```
sudo apt install openjdk-11-jdk
Verificar se foi bem instalado: java -version
Deverá aparecer algo assim:
"openjdk version "11.0.18" 2023-01-17
OpenJDK Runtime Environment (build 11.0.18+10-post-Raspbian-1deb11u1)
OpenJDK Server VM (build 11.0.18+10-post-Raspbian-1deb11u1, mixed mode)"
```
## 3- Instalação do Postgres:
### 3.1:Executar
```
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
	Aparecerá a seguinte mensagem: "Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
OK", ignorar...
```
### 3.2:Executar

```
RELEASE=$(lsb_release -cs)
echo "deb http://apt.postgresql.org/pub/repos/apt/ ${RELEASE}"-pgdg main | sudo tee  /etc/apt/sources.list.d/pgdg.list
```

### 3.3: Instalação do serviço postgres:

3.3.1: Executar:
```
 sudo apt update
```

3.3.2: Alguns websites relatam a necessidade de instalar o postgres-12, no entanto para o raspberry pi deve-se instalar o Postgres pré-definido:

```
sudo apt -y install postgresql
		
Iniciar o serviço postgres:  sudo service postgresql start
```
		
### 3.4: Depois de terminada a instalação do postgres, vamos criar um novo user e password:
3.4.1:
```
sudo su - postgres
psql

Aparecerá algo assim:
	psql (13.9 (Raspbian 13.9-0+deb11u1))
	Type "help" for help.
	postgres=# 
```
### 3.4.2: Executar:
```
 \password
  Aqui define-se a password, no meu caso escolhi "asd123"
	  
  Depois de definida a password, por favor digite "\q", em seguida fazer logout pelo "Ctrl + d"
```
		
### 3.4.3 Criação da Base de Dados do Thingsboard Edge:
```
psql -U postgres -d postgres -h 127.0.0.1 -W
Em seguida digite a password definida anteriormente;
```			   
### 3.4.4 Criação da db tb_edge:
```
CREATE DATABASE tb_edge;
```
### 3.4.5: Sair
```
\q
```

### 3.5:  Fazer o seguinte download:
```
wget https://github.com/thingsboard/thingsboard-edge/releases/download/v3.4.3/tb-edge-3.4.3.deb
```
		
### 3.5.1: Ir para a pasta onde está localizado o ficheiro anterior. No meu caso está "/home/pi"

	<a href='https://postimg.cc/1fkWd29s' target='_blank'><img src='https://i.postimg.cc/x1CWXVhJ/1.png' border='0' alt='1'/></a>

	3.5.2: Certificar que está na pasta do ficheiro (cd /home/pi) e executar:
			sudo dpkg -i tb-edge-3.4.3.deb
	
		   Deverá aparecer algo assim:
		   pi@raspberrypi:~ $ sudo dpkg -i tb-edge-3.4.3.deb
			A seleccionar pacote anteriormente não seleccionado tb-edge.
			(A ler a base de dados ... 109896 ficheiros e directórios actualmente instalados.)
			A preparar para desempacotar tb-edge-3.4.3.deb ...
			A adicionar o grupo `thingsboard' (GID 125) ...
			Concluído.
			A descompactar tb-edge (3.4.3EDGE-1) ...
			A instalar tb-edge (3.4.3EDGE-1) .
			
4:   Após se ter instalado o thingsboard local a correr no docker num computador.
	Cria-se uma instância edge no thingsboard versão docker conforme as próximas imagens:
	
	
	
	Neste passo por favor apontar os seguintes dois parâmetros
	EDGE_KEY=8790f310-be0b-45bf-269c-b925345a4ede
	EDGE_SECRET=kbratbzusppcnkjaxe1w
	
	4.1: A seguir é necessário editar o seguinte ficheiro:
	sudo nano /etc/tb-edge/conf/tb-edge.conf
	
	Neste ficheiro, necessita-se de alterar:
		export CLOUD_ROUTING_KEY = EDGE_KEY (apontadado anteriormente)
		export CLOUD_ROUTING_SECRET = EDGE_SECRET (apontadado anteriormente)
		
		export CLOUD_RPC_HOST=192.168.1.76 ( aqui coloca-se o endereço do computador que está a correr a imagem do thingsboard no docker)
		export CLOUD_RPC_PORT=7070 //descomentar esta linha
		
		export MQTT_BIND_PORT=11883 //descomentar esta linha
		
		Colocar adicionalmente a seguinte linha:
		
		export NETTY_MAX_PAYLOAD_SIZE=2000000
		
	4.2 Em seguida fazer login como root: sudo -i.
	
	4.3 Localizar o ficheiro install.sh que está neste caminho: "/usr/share/tb-edge/bin/install"
	
	4.4 Executar esse ficheiro : "./install.sh"
	
	4.5 Em seguida deverão aparecer algumas mensagens deste género:
	
	root@raspberrypi:/usr/share/tb-edge/bin/install# ./install.sh 
  ______    __      _                              ____                               __
 /_  __/   / /_    (_)   ____    ____ _   _____   / __ )  ____   ____ _   _____  ____/ /
  / /     / __ \  / /   / __ \  / __ `/  / ___/  / __  | / __ \ / __ `/  / ___/ / __  /
 / /     / / / / / /   / / / / / /_/ /  (__  )  / /_/ / / /_/ // /_/ /  / /    / /_/ /
/_/     /_/ /_/ /_/   /_/ /_/  \__, /  /____/  /_____/  \____/ \__,_/  /_/     \__,_/
                              /____/

 ===================================================
 :: ThingsBoard Edge CE ::       (v3.4.3EDGE)
 ===================================================

Starting ThingsBoard Edge Installation...
Installing DataBase schema for entities...
Installing SQL DataBase schema part: schema-entities.sql
Installing SQL DataBase schema indexes part: schema-entities-idx.sql
Installing SQL DataBase schema PostgreSQL specific indexes part: schema-entities-idx-psql-addon.sql
Installing DataBase schema for timeseries...
Installing SQL DataBase schema part: schema-ts-psql.sql
Successfully executed query: CREATE TABLE IF NOT EXISTS ts_kv_indefinite PARTITION OF ts_kv DEFAULT;
Loading system data...
Installation finished successfully!

Significando que a instalação foi realizada com sucesso.

	4.6 sudo service tb-edge restart
	
	4.7 Passados alguns segundos verificar o estado do serviço se está "active(running)":
	
		sudo service tb-edge status
		

5-          Através de um browser deverá aceder à porta 8080 do endereço de ip do Raspberry pi, conforme a imagem 2.

			As credenciais de acesso são:
				tenant@thingsboard.org e password: tenant
			
			Sucede que por vezes existe uma dessincronização entre o docker e o thingsboard edge.
			Nesse caso deverá ir à instância do Edge criada no passo 4 e pressionar o botão "Sync Edge"
			Após aparecer a mensagem Sync process start successfully", tente novamente fazer login. Caso persista o problema, reinicie o raspberry pi

