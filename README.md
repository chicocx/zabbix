# ZABBIX no Ubuntu 14.04

Como super usuário faça:

<pre>
add-apt-repository -y ppa:webupd8team/java

echo "deb http://apt.postgresql.org/pub/repos/apt/ trusty-pgdg main" >> /etc/apt/sources.list

wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add - 

apt-get update

apt-get -y install build-essential snmp vim libssh2-1-dev libssh2-1 libopenipmi-dev libsnmp-dev wget libcurl4-gnutls-dev fping libxml2 libxml2-dev curl libcurl3-gnutls libcurl3-gnutls-dev libiksemel-dev libiksemel-utils libiksemel3 libxml2 libxml2-dev ssh python-software-properties oracle-java8-installer oracle-java8-set-default apache2 php5 php5-pgsql postgresql-9.6 postgresql-client
libapache2-mod-php5 php5-gd php-net-socket libpq5 libpq-dev
</pre>

## Crie o banco de dados

<pre>
sudo -u postgres createdb zabbix
sudo -u postgres createuser -a -d -E -P zabbix
</pre>
senha: 123456

### Crie os dados

<pre>
cat zabbix-3.2.0/database/postgresql/schema.sql | psql -U zabbix zabbix
cat zabbix-3.2.0/database/postgresql/images.sql | psql -U zabbix zabbix
cat zabbix-3.2.0/database/postgresql/data.sql | psql -U zabbix zabbix
</pre>

## Configurações do apache

Edite o arquivo

<pre>
vim /etc/php5/apache2/php.ini
</pre>

E ajuste os parâmetros

<pre>
date.timezone = "America/Sao_Paulo"
max_execution_time = 300
max_input_time = 300
post_max_size = 16M
always_populate_raw_post_data = -1
</pre>

Adicione o usuário de sistema zabbix

## Configurações web

<pre>
adduser zabbix 
</pre>

Baixe o zabbix

<pre>
wget http://downloads.sourceforge.net/project/zabbix/ZABBIX%20Latest%20Stable/$VERSAO/zabbix-3.2.0.tar.gz
tar xzvf zabbix-3.2.0.tar.gz
cd zabbix-3.2.0
</pre)

Compile

<pre>
./configure --enable-server --enable-agent --enable-java --with-postgresql --with-net-snmp --with-jabber=/usr --with-libcurl=/usr/bin/curl-config --with-ssh2 --with-openipmi --with-libxml2
make install
</pre>

Faça o deploy do zabbix web

<pre>
mkdir /var/www/html/zabbix
cp -R frontends/php/* /var/www/html/zabbix/
chown -R www-data:. /var/www/html/zabbix/ 
</pre>

## Configurações gerais

Edite o arquivo
<pre>
vim /usr/local/etc/zabbix_agentd.conf
</pre>

conteúdo:

<pre>
PidFile=/tmp/zabbix_agentd.pid
LogFile=/tmp/zabbix_agentd.log
LogFileSize=2
DebugLevel=3
Server=127.0.0.1
ListenPort=10050
Hostname=ubuntu
Timeout=3
</pre>

Edite o arquivo
<pre>
vim  /usr/local/etc/zabbix_server.conf
</pre>

conteúdo:

<pre>
ListenPort=10051
LogFile=/tmp/zabbix_server.log
LogFileSize=2
PidFile=/tmp/zabbix_server.pid
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=123456
StartIPMIPollers=1
StartDiscoverers=5
Timeout=3
FpingLocation=/usr/bin/fping
</pre>


## Scripts de inicialização

Edite o arquivo

<pre>
vim  /etc/init.d/zabbix_server
</pre>

Conteúdo

<pre>
#!/bin/sh
#
# Zabbix daemon start/stop script.
#
# Written by Alexei Vladishev <alexei.vladishev@zabbix.com>.
NAME=zabbix_server
PATH=/bin:/usr/bin:/sbin:/usr/sbin:/home/zabbix/bin
DAEMON=/usr/local/sbin/${NAME}
DESC="Zabbix server daemon"
PID=/tmp/$NAME.pid
test -f $DAEMON || exit 0
set -e
case "$1" in
 start)
 echo "Starting $DESC: $NAME"
 start-stop-daemon --oknodo --start --pidfile $PID \
 --exec $DAEMON
 ;;
 stop)
 echo "Stopping $DESC: $NAME"
 start-stop-daemon --oknodo --stop --pidfile $PID \
 --exec $DAEMON
 ;;
 restart|force-reload)
 $0 stop
 sleep 3
 $0 start
 ;;
 *)
14
 N=/etc/init.d/$NAME
 echo "Usage: $N {start|stop|restart|force-reload}" >&2
 exit 1
 ;;
esac
exit 0
</pre>

Edite o arquivo

<pre>
vim /etc/init.d/zabbix_agentd
</pre>

Com o conteúdo

<pre>
#!/bin/sh
#
# Zabbix agent start/stop script.
#
# Written by Alexei Vladishev <alexei.vladishev@zabbix.com>.
NAME=zabbix_agentd
PATH=/bin:/usr/bin:/sbin:/usr/sbin:/home/zabbix/bin
DAEMON=/usr/local/sbin/${NAME}
DESC="Zabbix agent daemon"
PID=/tmp/$NAME.pid
test -f $DAEMON || exit 0
set -e
case "$1" in
 start)
 echo "Starting $DESC: $NAME"
 start-stop-daemon --oknodo --start --pidfile $PID \
 --exec $DAEMON
 ;;
 stop)
 echo "Stopping $DESC: $NAME"
 start-stop-daemon --oknodo --stop --pidfile $PID \
 --exec $DAEMON
 ;;
 restart|force-reload)
 $0 stop
 sleep 3
 $0 start
 ;;
 *)
 N=/etc/init.d/$NAME
 # echo "Usage: $N {start|stop|restart|force-reload}" >&2
 echo "Usage: $N {start|stop|restart|force-reload}" >&2
 exit 1
 ;;
esac
exit 0
</pre>

Torne os arquivos executáveis

<pre>
chmod +x /etc/init.d/zabbix_server /etc/init.d/zabbix_agentd
</pre>

Ainda, execute

<pre>
update-rc.d -f zabbix_server defaults
update-rc.d -f zabbix_agentd defaults
</pre>

## Inicialização de serviços

<pre>
service apache2 restart

/etc/init.d/zabbix_server start
/etc/init.d/zabbix_agentd start
</pre>


## Acesso

 

http://servidor/zabbix

<pre>
Usuario: Admin
Senha: zabbix
</pre>

## Máquina completa

http://franciscocalaca.com/downloads/zabbix.ova

<pre>
Usuário ubuntu: aluno
Senha ubuntu: 123456
</pre>
