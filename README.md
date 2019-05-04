# Configuração de Servidor Linux Projeto Udacity
## Projeto Final do Curso Full Stack Web Developer Nanodegree
### Descrição do Projeto
Fazer uma instalação de uma distribuição Linux em máquina virtual e prepará-la para hospedar aplicação web, para incluir instalação de updates, criar itens de segurança para alguns tipos de ataques e instalar e configurar servidor web Apache e servidor de banco de dados Postgresql.

- Endereço IP Estático: 3.215.229.252

- Porta SSH acessível: 2200

- URL da Aplicação: http://www.projetoudacity.tk

### Walkthrough

### Step 1: Iniciar uma Instância de um Servidor Linux Ubuntu na Amazon Lightsail

- Logar na [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources) usando uma conta Amazon Web Services.
- Depois de logado no site, clicar em `Create instance`. 
- Escolha plataforma `Linux/Unix`, `OS Only` e `Ubuntu 16.04 LTS`.
- Escolha um plano (Peguei o mais barato, $3.5/month).
- Se você apagar a instância antes do primeiro mês, não será cobrado.
- Mantenha o nome sugerido pela AWS ou troque por um de sua escolha.
- Clique no botão `Create` para criar a instância.
- Espere pela instância iniciar.

**Referência**
- ServerPilot, [How to Create a Server on Amazon Lightsail](https://serverpilot.io/community/articles/how-to-create-a-server-on-amazon-lightsail.html).

### Step 2: SSH no servidor

- Clique no menu `Account` no Amazon Lightsail, depois clique em `SSH keys` e faça o download da chave criada no servidor.
- Mova o arquivo da chave com extensão ".pem" para uma pasta de sua escolha em sua máquina local.
- Agora abra uma caixa de terminal se estiver no linux. Se estiver no windows, abra uma caixa de terminal do Git Bash. Para instalar [Git Bash](https://gitforwindows.org/).
- Isso porque vamos usar o comando ssh para conectar no servidor.
- Digite:
- `ssh ubuntu@IP_DO_SERVIDOR -i ARQUIVO_DE_CHAVE.pem`
- Substitua IP_DO_SERVIDOR pelo IP público do Lightsail. 
- Substitua ARQUIVO_DE_CHAVE.pem pelo nome que vc escolheu ao fazer o download do arquivo de chave do servidor.

### Step 3: Atualize os pacotes instalados
No terminal digite:
```
sudo apt-get update
sudo apt-get upgrade
```
### Step 4: Instale o servidor Apache e a biblioteca wsgi
No terminal digite:
```
sudo apt-get install apache2 libapache2-mod-wsgi-py3
```
Nesse momento, com o apache instalado é pra vc conseguir digitar o IP público do servidor no browser e conseguir ver uma página do Ubuntu.


### Step 5: Clonando Projeto 
No terminal digite:
```
sudo apt-get install python3-git
cd /var/www
sudo git clone https://github.com/tetigo/ud_ProjetoCatalogo.git myApp
```


### Step 6: Instale o ambiente virtual

Como utilizo o python3 no projeto, criaremos um ambiente virtual onde vamos instalar o python 3 e depois mandar apontar para essa instalação.
No terminal digite:
```
sudo apt-get install python-virtualenv
sudo apt-get install python3-pip
```

### Step 7: Criando o Ambiente Virtual
Verifique que inda está dentro do diretório:
/var/www/myApp

Para isso no terminal digite:
`pwd`
Se não estiver dentro desse diretório, digite no terminal:
```
cd /var/www/myApp
```
Agora estando dentro desse diretório digite no terminal:
`sudo virtualenv -p python3 venv3`
Agora ative esse ambiente:
`. venv3/bin/activate`

**References**
- Flask documentation, [virtualenv](http://flask.pocoo.org/docs/0.12/installation/).
- [Create a Python 3 virtual environment](https://superuser.com/questions/1039369/create-a-python-3-virtual-environment).


### Step 8: Habilite o Python3
No terminal digite:
```
cd /etc/apache2/mods-enabled
sudo nano wsgi.conf 
```
Nesse arquivo que abriu procure pela linha:
#WSGIPythonPath directory|directory-1:directory-2:...
Digite na linha abaixo dela:
`WSGIPythonPath /var/www/myApp/venv3/lib/python3.5/site-packages`

### Step 9: Instalando Pacotes

Volte para o diretorio myApp: `cd /var/www/myApp`
Verifique que ainda está com o ambiente virtual ativado: #(venv3)

Agora execute os comandos:
```
sudo apt-get install python3-flask
sudo apt-get install python3-pip
sudo pip3 install -r requirements.txt
```


### Step 10: Criar arquivo wsgi
Ainda dentro da pasta /var/www/myApp entre o comando:
`sudo nano myApp.wsgi`
Cole o seguinte:
```
import sys
sys.path.insert(0, "/var/www/myApp")
from main import app as application
application.secret_key = 'supersecretkey'
```
Saia do arquivo e salve: Ctrl + X, yes, Enter

### Step 11: Criar Arquivo de Configuração do Apache
`cd /etc/apache2/sites-available`
Executar:
`sudo nano myApp.conf`

Colar nesse Arquivo:
```
<VirtualHost *>
 ServerName example.com
 WSGIScriptAlias / /var/www/myApp/myApp.wsgi
 WSGIDaemonProcess hello
 <Directory /var/www/myApp>
  WSGIProcessGroup hello
  WSGIApplicationGroup %{GLOBAL}
   Order deny,allow
   Allow from all
 </Directory>
 ErrorLog ${APACHE_LOG_DIR}/error.log
 LogLevel warn
 CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
 ```
Agora, desabilite o site padrão do Apache, habilite sua aplicaçã Flask, e reinicie o apache para que as mudanças tenham efeito.
Execute os seguintes comandos:
```
sudo a2dissite 000-default.conf
sudo a2ensite myApp.conf
sudo service apache2 reload
```

**Resources** 
- [Getting Flask to use Python3 (Apache/mod_wsgi)](https://stackoverflow.com/questions/30642894/getting-flask-to-use-python3-apache-mod-wsgi)
- [Run mod_wsgi with virtualenv or Python with version different that system default](https://stackoverflow.com/questions/27450998/run-mod-wsgi-with-virtualenv-or-python-with-version-different-that-system-defaul)


### Step 12: Autenticar o Login através do Google
Ir em [Google Cloud Plateform](https://console.cloud.google.com/).
Clique em APIs & services no menu esquerdo.
Clique em Credentials.
Crie um OAuth Client ID (na tab Credentials), e coloque http://3.215.229.252 e http://projetoudacity.tk e www.projetoudacity.tk como origens Javascript autorizadas.
Coloque http://projetoudacity.tk/gCallback e http://www.projetoudacity.tk/gCallback como URI de redirecionamento autorizadas. Faça o download do arquivo JSON, abra e copie o valor do CLIENT_ID e do CLIENT_SECRET.
Esses valores vão ser usados para gerar as variáveis de ambiente usadas na aplicação.

**Resources**
- [Google Authentication with Python and Flask](https://www.mattbutton.com/2019/01/05/google-authentication-with-python-and-flask/)
- [Add Google Oauth2 login in your flask web app](https://medium.com/@bittu/add-google-oauth2-login-in-your-flask-web-app-9f455695341e)


### Step 13: Criar arquivo  .flaskenv
Esse arquivo contem as variáveis de ambiente.
Dentro do mesmo diretório myApp, execute:
`sudo nano .flaskenv`
Cole o seguinte dentro e salve:
```
CLIENT_ID='872724505668-314ii0tf7hagec22nk1sc78c38d7hbec.apps.googleusercontent.com'
CLIENT_SECRET='ZGZ_HLd3gw0PRvKDgNqiVo2K'
SECRET_KEY='SuperChaveSecreta'

```
Testei algumas vezes e não sei por qual motivo, às vezes não carregava essas variáveis com a biblioteca `dotenv`.
Verifique com o comando `env` se essas variaveis se encontram entre as variaveis de ambiente. Caso negativo, edite o arquivo `config.py` e carregue as variaveis com os respectivas valores. Para isso, comente as respectivas linhas do arquivo `config.py` e cole as linhas mostradas acima dentro do arquivo.

### Step 14: Instalando Banco de Dados
No terminal execute:
```
sudo apt-get install postgresql
sudo apt-get update
sudo apt-get upgrade
```

Criando o Banco
Execute:
`sudo su - postgres`
Agora que está com o usuario postgres ativo, digite o comando:
`psql`

Agora o prompt de comando se parece com: `postgres=#`
execute os comandos:
```

CREATE USER catalog WITH PASSWORD 'catalog';
ALTER USER catalog CREATEDB;
CREATE DATABASE catalog WITH OWNER catalog;
```
Conecte à base de dados chamada catalog com o comando:
`\c catalog`
Agora o prompt de comando se parece com: `catalog=#`
Execute o seguinte:
```
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
```
Saia do postgres e usuario postgres com os comandos:
```
\q
exit
```
Agora precisamos atualizar a conexão do nosso programa que está atualmente apontando para um banco sqlite e agora queremos que ele aponte para um banco postgresql.
No programa `super.py` substitua `sqlite://banco.sqlite3` por `postgresql://catalog:catalog@localhost/catalog` 

### Step 15: Inicializando o Banco de Dados
Execute no terminal:
```
sudo apt-get install python3-psycopg2
sudo flask db init
sudo flask db migrate
sudo flask db upgrade
sudo python3 carga_banco.py
```
Com os comandos acima, instalamos biblioteca psycopg2 para conseguir rodar o Migrate.
Executamos os 3 comandos do Migrate para gerar as tabelas automaticamente.
Carregamos o banco com dados iniciais.

### Step 16: Criar IP Estático no Servidor
Isso serve para que o endereço IP do servidor não mude a cada vez que inicializamos o servidor.
Assim podemos usar esse IP estático para atrelar a um endereço de domínio externo para facilitar a navegação até nossa aplicação e também para integrar com a plataforma Google.
Para isso, na página do Lightsail clique em:
Home, depois Network, depois no botão criar IP Estático.

### Step 17: Criar Nome de Domínio 
criar nome dominio em Freenom pois é gratis.
Também é auto-explicativo, muito fácil.
https://www.freenom.com/pt/index.html?lang=pt

No site freenom, vc deve atrelar o nome de domínio que vc escolheu a um endereço de DNS. Esse endereço é o IP estático criado no Step 14 acima.
Isso vai facilitar nossa vida na hora de integrar a aplicação com a plataforma Google, especialmente na hora de colocar endereço de Callback.


### Step 18: Alterar Portas
Mudar porta 22 para 2200
```
sudo nano /etc/ssh/sshd_config
```
Mude a pora de 22 para 2200 na linha 5.
Salve e saia usando CTRL+X e confirme com Y.
Restart SSH: 
`sudo service ssh restart`

### Step 19: Configurando Firewall
Configure o firewall default do Ubuntu para somente permitir conexões para SSH (port 2200), HTTP (port 80), and NTP (port 123).
Execute os comandos no terminal:
```
sudo ufw status                  
sudo ufw default deny incoming   
sudo ufw default allow outgoing  
sudo ufw allow 2200/tcp          
sudo ufw allow www              
sudo ufw allow 123/udp      
sudo ufw deny 22            
sudo ufw enable
```
Agora verificando o status:
`sudo ufw status`. 
A saída deve ser:
```
Status: active
To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere                 
80/tcp                     ALLOW       Anywhere                 
123/udp                    ALLOW       Anywhere                 
22                         DENY        Anywhere                 
2200/tcp (v6)              ALLOW       Anywhere (v6)            
80/tcp (v6)                ALLOW       Anywhere (v6)            
123/udp (v6)               ALLOW       Anywhere (v6)            
22 (v6)                    DENY        Anywhere (v6)
 ```
Sair da conexão SSH:
`exit`

No site do Lightsail, em home, na instância que está rodando tem 3 bolinhas na instância. Clicar nelas, depois em Manage.
Então clicar em Networking e mudar as configurações do firewall para bater com as informações de firewall que acabamos de digitar acima. 
Permitir portas 80(TCP), 123(UDP), and 2200(TCP), e negar a porta padrão 22.


### Step 20: Usar Fail2Ban para banir atacantes
Software que protege contra ataques de força-bruta.
Executar no terminal:
`sudo apt-get install fail2ban`
Instale sendmail:
`sudo apt-get install sendmail iptables-persistent`
Execute:
`sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
Mude as configurações abaixo no arquivo: `/etc/fail2ban/jail.local`
```
set bantime = 600
destemail = seu_email@seu_dominio
action = %(action_mwl)s 
Debaixo da chave [sshd] mude port = ssh por port = 2200.
Restart o serviço: 
sudo service fail2ban restart
```

### Step 21: Instalar Updates Automaticamente
Execute:
`sudo apt-get install unattended-upgrades`

Edit /etc/apt/apt.conf.d/50unattended-upgrades, descomente a linha: 
`${distro_id}:${distro_codename}-updates` e salve.

Modifique /etc/apt/apt.conf.d/20auto-upgrades :
```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
```
Habilite: 
`sudo dpkg-reconfigure --priority=low unattended-upgrades`
Restart Apache: 
`sudo service apache2 restart`

### Step 22: Update dos pacotes para versões recentes
```
sudo apt-get update
sudo apt-get dist-upgrade
sudo shutdown -r now
```
conectar novamente substituindo ip e xx.pem: 
`ssh ubuntu@ip -i xx.pem -p 2200`

### Step 23: Cirar novo usuário chamado  grader

Enquanto logado como ubuntu, execute:
`sudo adduser grader`
Entre password 2 vezes quando pedido.
A senha que utilizei foi `grader`.
Editar as permissões de grader:
`sudo visudo`
Procurar pela linha:
`root    ALL=(ALL:ALL) ALL`
Abaixo desta linha coloca:
grader   ALL=(ALL:ALL) ALL

Salvar e sair.


### Step 24: Criar nova key
Na maquina local rodar:
`ssh-keygen` dentro de pasta .ssh criada dentro do seu usuario.
Dar um nome para o arquivo de senha.
Vão ser criados 2 arquivos sendo um sem extensao e um .pub
rodar `cat arq.pub` e copiar conteudo.

### Step 25: Configurar .ssh de grader
logar novamente no servidor com ubuntu .....
entrar no usuario grader:
`su - grader`
entrar senha de grader
rodar pwd para verificar a pasta atual em que se encontra
vai ser `home/grader`
cria aqui uma pasta .ssh
sudo mkdir .ssh
cd .ssh
rodar pwd
agora deve ser `home/grader/.ssh`
criar arquivo authorized_keys:
`sudo nano authorized_keys`
colar o conteudo previamente copiado da chave publica criada na máquina local.
sair da pasta .ssh:
`cd ..`
Dar permissões: 
`chmod 700 .ssh`
`chmod 644 .ssh/authorized_keys`
Checa se PasswordAuthentication está setado com `no` no arquivo:
`/etc/ssh/sshd_config`
Restart SSH: `sudo service ssh restart`
Na máquina local rodar: 
`ssh grader@3.215.229.252 -i ~/.ssh/grader_key -p 2200`
Agora vc está logado com o usuário grader.

### Step 26: Configurar o Timezone Local para UTC
Enquanto logado como `grader`, configure o TimeZone:
`sudo dpkg-reconfigure tzdata` 
Selecione none of the above
Selecione UTC
**References**
- Ubuntu Wiki, [UbuntuTime](https://help.ubuntu.com/community/UbuntuTime)

### Step 27: Últimas Atualizações 
```
sudo apt-get update
sudo apt-get upgrade
```


- DigitalOcean [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
