Tercer Parcial Sistemas operativos

Universidad ICESI

Christian D. Cardenas Camargo

A00212740

url git: www.github.com/cdcardenas/so-exam3


1. para la instalacion del stack de sensu se ejecutaron los siguientes comandos

  $ echo '[sensu]
  
  $ name=sensu
  
  $ baseurl=https://sensu.global.ssl.fastly.net/yum/$releasever/$basearch/
  
  $ gpgcheck=0

  $ enabled=1' | sudo tee /etc/yum.repos.d/sensu.repo
