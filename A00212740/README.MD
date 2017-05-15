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



     

     

  
