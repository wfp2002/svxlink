#SVXLINK (Echolink Linux) 

Instalando Echolink no Linux (SVXLINK)

sudo apt-get install g++ make libsigc++-1.2-dev libgsm1-dev libpopt-dev tcl8.5-dev libgcrypt-dev libspeex-dev libasound2-dev alsa-utils libqt4-dev git-core

Se der erro no comando abaixo como add-apt not found instale o seguinte:

sudo apt-get install software-properties-common python-software-properties

sudo add-apt-repository ppa:felix.lechner/hamradio

sudo apt-get update

sudo apt-get install svxlink-server

Echolink Client Se precisar

sudo apt-get install qtel

Arquivos para Editar

sudo nano /etc/svxlink/svxlink.conf 

sudo nano /etc/svxlink/svxlink.d/ModuleEchoLink.conf 

Diretorios de SOM

/usr/share/svxlink/sounds/en_US/
