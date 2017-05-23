Tercer Parcial Sistemas operativos

Universidad ICESI

Christian D. Cardenas Camargo

A00212740

url git: www.github.com/cdcardenas/so-exam3


Para la instalacion del stack de sensu se ejecutaron los siguientes comandos para la configuracion de la maquina cliente

     echo '[sensu]
     name=sensu
     baseurl=https://sensu.global.ssl.fastly.net/yum/$releasever/$basearch/
     gpgcheck=0
     enabled=1' | sudo tee /etc/yum.repos.d/sensu.repo
luego se procede a descargas e installar sensu mediante el gestor de paquetes yum

    yum install sensu -y
    sensu-install -p sensu-plugin

posteriormente se debe instalar el servicio httpd, y configurar rabbitMQ, que es un broked de protocolo M2M

para configurar rabbitMQ se debe configurar un archivo json con el nomble client.json y otro con el nombre rabbitMQ.json
La estructura de los archivos es la siguiente respectivamente

    {
    "client": {
      "name": "A00317220",
      "address": "192.168.57.4",
      "subscriptions": ["webservers"]
     }
    }

     {
     "rabbitmq": {
       "host": "192.168.57.3",
       "port": 5672,
       "vhost": "/sensu",
       "user": "sensu",
       "password": "password",
       "heartbeat": 10,
       "prefetch": 50
      }
     }


en la siguiente dirección

    cd /etc/sensu/conf.d

para instalar el servicio httpd ejecutamos el comando

    yum install httpd -y
    
Luego como queremos que sensu quede instalado de la forma mas completa posible, procedemos a instalar un conjunto de plugins que extienden su funcionalidad, por medio del comando 

     cd /etc/sensu/plugins
     
 el cual es el directorio de plugins de sensu, despues se crea el siguiente script de ruby
 
     #!/usr/bin/env ruby

     procs = `ps aux`
     running = false
     procs.each_line do |proc|
          running = true if proc.include?('httpd')
     end
     if running
          puts 'OK - Apache daemon is running'
          exit 0
     else
          puts 'WARNING - Apache daemon is NOT running'
          exit 1
     end

Ahora, para la configuracion de la maquina Servidor se realizaron los siguientes pasos

     echo '[sensu]
     name=sensu
     baseurl=https://sensu.global.ssl.fastly.net/yum/$releasever/$basearch/
     gpgcheck=0
     enabled=1' | sudo tee /etc/yum.repos.d/sensu.repo
     
luego se debe instalar sensu, plugins, erlang, socat, activar el servicio redis y configurar rabbitMQ

instalacion sensu en el servidor:

     yum install sensu -y
     sensu-install -p sensu-plugin
     sensu-install -p sensu-plugins-slack
     su -c 'rpm -Uvh http://download.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-9.noarch.rpm'
     
instalacion socat:

     yum install socat -y

instalacion erglang:

     yum install erlang -y
 
Activacion redis

     service redis start

Configuracion rabbitMQ server

     su -c 'rpm -Uvh http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.9/rabbitmq-server-3.6.9-1.el7.noarch.rpm'
     service rabbitmq-server start
     rabbitmqctl add_vhost /sensu
     rabbitmqctl add_user sensu password
     rabbitmqctl set_permissions -p /sensu sensu ".*" ".*" ".*"
     rabbitmq-plugins enable rabbitmq_management
     chown -R rabbitmq:rabbitmq /var/lib/rabbitmq
     
Configuracion de sesion rabbitMQ
     
     rabbitmqctl add_user test test
     rabbitmqctl set_user_tags test administrator
     rabbitmqctl set_permissions -p / test ".*" ".*" ".*"
     
Para el despliegue de la informacion usaremos el dashboard uchiwa que se instala mediante el suguiente comando

     yum intall uchiwa -y
     
Como es costumbre, se debe verificar que los puertos 5672, 15672 y 3000 esten abiertos, en caso de que no lo esten los abrimos mediante los comandos:

     firewall-cmd --zone=public --add-port=5672/tcp --permanent
     firewall-cmd --zone=public --add-port=15672/tcp --permanent
     firewall-cmd --zone=public --add-port=3000/tcp --permanent
     firewall-cmd --reload
     
finalmente para asegurarnos que los servicios tomen los cambios de red debemos reiniciarlos

     service sensu-server restart
     service sensu-api restart
     service uchiwa restart
     
Funcionamiento RABBITMQ y Sensu

![][1]
![][2]


Funcionamiento Uchiwa

![][3]
![][4]

Para la evidencia de caida del servicio se utilizo la herramienta de creacion de gif para la captura de un corto video que evidencia la caida de un servicio y su alerta respectiva mediante Uchiwa dashboard

![][5]


7 para la instalacion de stack ELK(elasticsearch, logstash, kibana, filebeat en el client)) se utilizaron los siguientes comandos:

     yum install java-1.8.0-openjdk.x86_64

--ELASTICSEARCH--

con ese comando se instala el jdk 1.8.0 de java y posteriormente se debe instalar y configurar la llave publica de elasticsearch

     rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
   
luego en la siguiente direccion creamos un archivo .repo que contendra el repositorio de elasticsearch

     vi /etc/yum.repos.d/elasticsearch.repo
     
luego en este archivo escribimos:

     [elasticsearch-5.x]
     name=Elasticsearch repository for 5.x packages
     baseurl=https://artifacts.elastic.co/packages/5.x/yum
     gpgcheck=1
     gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
     enabled=1
     autorefresh=1
     type=rpm-md
     
Ahora que ya tenemos el repositorio listo. procedemos a instalar elasticsearch

     yum install elasticsearch -y
     
--LONGSTASH--

como se hizo para elasticsearch, para longstash debemos crear un archivo que contenga el repositorio de longstash

     vi /etc/yum.repos.d/logstash.repo
     
y escribimos en el documento:

     [logstash-5.x]
     name=Elastic repository for 5.x packages
     baseurl=https://artifacts.elastic.co/packages/5.x/yum
     gpgcheck=1
     gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
     enabled=1
     autorefresh=1
     type=rpm-md

guardamos el achivo con :x e instalamos longstash mediante el comando:

     yum install logstash -y

--INSTALACIÓN KIBANA--

siguiendo la misma estructura de los anteriores servicios del stack ELK

     vi /etc/yum.repos.d/kibana.repo
     
dentro del editor escribimos:

     [kibana-5.x]
     name=Kibana repository for 5.x packages
     baseurl=https://artifacts.elastic.co/packages/5.x/yum
     gpgcheck=1
     gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
     enabled=1
     autorefresh=1
     type=rpm-md

instalamos Kibana con :

     yum install kibana -y
     
--FILEBEAT--


Llave publica:

     sudo rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch
     
repositorio:

     vi /etc/yum.repos.d/elastic.repo
     
     [elastic-5.x]
     name=Elastic repository for 5.x packages
     baseurl=https://artifacts.elastic.co/packages/5.x/yum
     gpgcheck=1
     gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
     enabled=1
     autorefresh=1
     type=rpm-md

para terminar, instalamos 

     sudo yum install filebeat -y
     
     
     
     
     
Para verificar el correcto funcionamiento del stack, se verifico usando el ejemplo 0 en el gist del profesor Daniel Barragán, ubicado en la siguiente url: gist.github.com/d4n13lbc/be1ad5039dff1c058b335482488d4965

Abrimos los siguientes puertos:

     firewall-cmd --zone=public --add-port=9200/tcp --permanent
     firewall-cmd --reload

para la prueba con este ejemplo se debe editar la configuracion del archivo elasticsearch.yml

     vi /etc/elasticsearch/elasticsearch.yml
     
 agregamos los siguientes cambios:
 
     # Set the bind address to a specific IP (IPv4 or IPv6):
     network.host: 192.168.57.5
     # Set a custom port for HTTP:
     http.port: 9200
     
     
Para Kibana tambien debemos, editar la configuración y abrir los siguientes puertos:

     firewall-cmd --zone=public --add-port=5601/tcp --permanent
     firewall-cmd --reload

Editamos el siguiente archivo:

     vi /etc/kibana/kibana.yml
     
agregamos los siguientes cambios

     # Kibana is served by a back end server. This setting specifies the port to use.
     server.port: 5601
     # To allow connections from remote users, set this parameter to a non-loopback address.
     server.host: "192.168.57.5"
     # The URL of the Elasticsearch instance to use for all your queries.
     elasticsearch.url: "http://192.168.57.5:9200

Ahora para la prueba iniciamos Kibana y elasticsearch

     service kibana start
     service elasticsearch start
     
 
Pruebas de funcionamiento:

![][6]
![][7]




[1]: images/rabbit.png
[2]: images/rabbit2.png
[3]: images/uchiwa.png
[4]: images/uchiwa2.png
[5]: images/parcial3GrupoChristianJorgeyRay.gif
[6]: images/datos.png
[7]: images/kibana.png
