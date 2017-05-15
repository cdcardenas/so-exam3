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

para configurar rabbitMQ se debe configurar un archivo json con el nomble client.json

    {
    "client": {
    "name": "A00317220",
    "address": "192.168.57.4",
    "subscriptions": ["webservers"]
     }
    }


en la siguiente dirección

    cd /etc/sensu/conf.d

para instalar el servicio httpd ejecutamos el comando

    yum install httpd -y
    
  
