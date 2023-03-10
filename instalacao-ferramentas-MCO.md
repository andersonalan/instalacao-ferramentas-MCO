# Tutorial para instalação de Java 8 / Tomcat 9 / PostgreSQL

Tutorial básico para instalação das ferramentas necessárias para a instalação do sistema MCO no ambiente `Linux`

Requisitos básicos necessários:
- Disponibilidade de um servidor `Linux Ubuntu` (independente da versão);
- Acesso ao servidor `Linux Ubuntu` via o comando `ssh` ou diretamente do servidor; 

Após iniciar o servidor `Ubuntu` e executar o comando `apt update` para atualizar a lista de pacotes
```
sudo apt update
```
Em seguida atualizar os pacotes
```
sudo apt upgrade -y
```
Finalizado a etapa anterior é possível que seja solicitado o reinicio do serviço.

___

## Primeiro passo - Instalando PostgreSQL

Criar a configuração do repositório de arquivos:
```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
```
Importar a chave de assinatura do repositório:
```
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
```
Atualizar as listas de pacotes:
```
sudo apt update
``` 
Instalar a versão mais recente do `PostgreSQL`:
```
sudo apt -y install postgresql
```
  
Realizar o ajuste de memória e conexões dentro do arquivo `postgresql.conf` e no arquivo `pg_hba.conf`, respectivamente.
___
### Ajuste de memória e conexões PostgreSQL 

#### Ajuste de memória

Ajustar as configurações de memória do **postgres**, acessando a calculadora de configuração através do link: [PGTune](https://pgtune.leopard.in.ua/ "calculadora de ajuste de memória PostgreSQL"), inserindo os parâmetros da máquina.

> Inserindo os parâmetros da máquina, o site calculará os valores que serão alterados dentro do arquivo `postgresql.conf`.
>
> Como exemplo utilizei os parâmetros abaixo, extraindo a partir da VM do ubuntuserver.

Parameters of your system
- DB version = 15
- OS Type = Linux
- DB Type = Mixed type of application
- Total Memory (RAM) = 2GB
- Number of CPUs = 2
- Number of Connections = 100
- Data Storage = SSD storage

Clicando no botão `Generate` a calculadora irá mostrar os parâmetros ao lado direito a serem utilizados.
A seguir abrir arquivo `postgresql.conf` localizado em `/etc/postgresql/15/main/postgresql.conf` e através do programa `nano` realizar a alteração dos parâmetros com o comando:

>lembrando de estar em usuário root
```
sudo su
```
```
nano /etc/postgresql/15/main/postgresql.conf
```
> Abaixo o resultado do exemplo:

- max_connections = 100
- shared_buffers = 512MB
- effective_cache_size = 1536MB
- maintenance_work_mem = 128MB
- checkpoint_completion_target = 0.9
- wal_buffers = 16MB
- default_statistics_target = 100
- random_page_cost = 1.1
- effective_io_concurrency = 2
- work_mem = 1310kB
- min_wal_size = 1GB
- max_wal_size = 4GB

>Com o aquivo `postgres.conf` aberto utilize o **Ctrl W** para localizar os parâmetros.  
>Caso o parâmetro esteja comentado com hashtag `#` remover a hashtag `#`.  
>Finalizadas as alterações, executar o comando **Ctrl O** e **Enter** para salvar e **Ctrl X** para fechar.  
___

#### Ajuste de conexões

Criar uma `senha de acesso` ao **postgres** e acessar o arquivo `pg_hba.conf` utilizando o programa `nano`:

Acessar o `prompt Postgres` com o comando abaixo.
```
sudo -u postgres psql
```
Dentro do `terminal Postgres` criar a senha do usuário **postgres** com o seguinte comando:

```
alter user postgres with password 'senha desejada';
```
Sair do `prompt Postgres` com o comando `\q`:
```
\q
```
Acessar a pasta `pg_hba.conf` através do programa `nano` utilizando o comando a seguir:
```
nano /etc/postgresql/15/main/pg_hba.conf 
```

>Com o arquivo aberto localizar a linha "Database administrative login by Unix domain socket" e alterar o campo da linha abaixo de peer para scram-sha-256   
>Conforme o exemplo abaixo:  
>local all postgres scram-sha-256  
>
>E localizar a linha abaixo da linha "Unix domain socket connections only"  
>local all all scram-sha-256

>Finalizadas as alterações, executar o comando **Ctrl O** e **Enter** para salvar e **Ctrl X** para fechar. 

Após realizar a alteração de conexão necessário reiniciar o **postgres** utilizando o **systemctl restart**:
```
systemctl restart postgresql
```
A partir de agora para acessar o **PostgreSQL** será necessário inserir a senha anteriormente criada.  
Utilizar o comando abaixo e em seguida a insira a senha:
```
psql -U postgres --password
```
Caso o acesso ao prompt do postgres ocorra, então as alterações foram um sucesso, caso contrário, refaça os passos acima.  
Sair do `prompt Postgres` com o comando `\q`:
```
\q
```
A instalação do PostgreSQL está completa!
___

## Segundo passo - Instalando Java 8

Antes de iniciar a instalação atualize o gerenciador de pacotes com o comando:
```
sudo apt update
```
Instalar o Java Development Kit (JDK), através do comando a seguir:
```
sudo apt install openjdk-8-jdk-headless -y
```
Para confirmar a instalação executar os comandos `java -version` e `javac -version`.

A instalação do Java 8 está completa!

___

## Terceiro passo - Instalando Tomcat 9

Antes de iniciar a instalação atualizar o gerenciador de pacotes com o comando:
```
sudo apt update
```
Criar o diretório de instalação do tomcat através dos comandos a seguir.
>Lembrando de estar em modo root.
```
mkdir -p /applications/installers
```
Acessar o diretório criado através do comando:
```
cd /applications/installers/
```
Baixar o instalador `.tar.gz` através do comando `wget` junto ao link do download:  

>Lembrando que precisa estar na pasta `/applications/installers` para baixar o instalador.

```
wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.21/bin/apache-tomcat-9.0.21.tar.gz
```
Extrair o tomcat no diretório criado com o comando:
```
tar -xvzf apache-tomcat-9.0.21.tar.gz
```
Alterar o nome do arquivo:
>Como exemplo utilizei o nome "tomcat"
```
mv /applications/installers/apache-tomcat-9.0.21 /applications/installers/tomcat 
```
Para o próximo passo será necessário saber o caminho do `java 8` e para isso utilizar o comando a seguir:
```
update-alternatives --config java
```
Criar o arquivo `tomcat.service` necessário para a inicialização do gerenciador de serviços `systemctl` dentro da pasta `/etc/systemd/system` através do seguinte comando:
```
nano /etc/systemd/system/tomcat.service
```
Abaixo segue o modelo do arquivo, `copiar` o modelo e `colar` dentro do `tomcat.service` pelo programa `nano`:
>Lembrando que caso o caminho das variáveis de ambiente esteja diferente precisa alterar o caminho no arquivo.
```
[Unit]
Description=Apache Tomcat Web Application Container
Wants=network.target
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/jre/

Environment=CATALINA_PID=/applications/installers/tomcat/temp/tomcat.pid  
Environment=CATALINA_HOME=/applications/installers/tomcat/
Environment='CATALINA_OPTS=-Xms512M -Xmx1G -Djava.net.preferIPv4Stack=true'
Environment='JAVA_OPTS=-Djava.awt.headless=true'

ExecStart=/applications/installers/tomcat/bin/startup.sh
ExecStop=/applications/installers/tomcat/bin/shutdown.sh
SuccessExitStatus=143

RestartSec=10
Restart=always
 
[Install]
WantedBy=multi-user.target
```
>Finalizando as alterações no arquivo, **Ctrl O** e **Enter** para salvar e **Ctrl X** para fechar.

Executar o `systemctl` para habilitar o serviço de inicialização, e para isso utilizar o comando `enable` e depois iniciar com o comando `start`.  
Utilizar o `systemctl daemon-reload` para atualizar as informações do `tomcat.service`.
```
systemctl daemon-reload
```
Habilitar o tomcat via `enable` utilizando o comando:
```
systemctl enable tomcat
```
Depois de habilitado utilizar o comando `start` para iniciar o `tomcat`:
```
systemctl start tomcat
```
Tomcat instalado e serviço de inicialização criado com sucesso!  

Para verificar o status do serviço utilize o comando `status`:
```
systemctl status tomcat
```
 
Todas as ferramentas necessárias para a instalação do sistema MCO foram instaladas com sucesso!
