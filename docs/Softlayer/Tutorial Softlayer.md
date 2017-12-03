# Creacion de ambientes de máquinas virtuales en IBM Cloud (Softlayer/Bluemix)

Para crear varias máquinas virtuales (p.e. para simular peers de varias organizaciones) en IBM Cloud, conviene usar la herramienta de línea de comando en vez de usar la interfaz web.

En este documento describimos

## Instalar cliente de linea de comando

Debo tener python y pip instalado. La instalación depende del sistema operativo. Luego, la instalación es `pip install softlayer`.

A veces es necesario usar un proxy, en ese caso en windows hago:

````
set http_proxy=http://ip:puerto
set https_proxy=http://ip:puerto
````

En Linux hago:

````
export http_proxy=http://ip:puerto
export https_proxy=http://ip:puerto
````

Ahora ya puedo usar slcli. Por ejemplo, miro la configuración:

````
slcli config show
:..............:..................................................................:
:         name : value                                                            :
:..............:..................................................................:
:     Username : ...                                                              :
:      API Key : ...                                                              :
: Endpoint URL : https://api.softlayer.com/xmlrpc/v3.1                            :
:      Timeout : not set                                                          :
:..............:..................................................................:
````

Algunos de estos parámetros están incorrectos y hay que cambiarlos por valores apropiados. De eso se trata la siguiente parte de este documento.

## Obtener clave y usuario para conectarse a Softlayer

Los parámetros que precisa `slcli` para funcionar, o sea, para conectarse a la API de Softlayer, son un usuario y una api key.

En este documento dice como generarla, o, si ya está generada, como verla:
https://knowledgelayer.softlayer.com/procedure/generate-api-key

Luego ejecuto `slcli config setup` ingresando el usuario y la clave cuando me la solicite.

Para ver si quedó todo bien hago:

````
> slcli vs list
:..........:..........:............:..............:............:........:
:    id    : hostname : primary_ip :  backend_ip  : datacenter : action :
:..........:..........:............:..............:............:........:
: 45329897 :  prueba  :     -      : 10.55.143.14 :   sjc01    :   -    :
: 45330005 : prueba2  :     -      : 10.55.143.7  :   sjc01    :   -    :
:..........:..........:............:..............:............:........:
````

## Creacion de máquinas virtuales

````
slcli vs create --hostname=prueba --domain=prueba.com --cpu 2 --memory 1024 -o UBUNTU_16_64 --datacenter=sjc01 --billing=hourly --public --private
````

Hay otros paráemtros relevantes, p.e. para instalarle software puedo poner los comandos de instalación en un archivo y subirlo con:

````
  -i, --postinstall TEXT          Post-install script to download
  -k, --key TEXT                  SSH keys to add to the root user (multiple
                                  occurrence permitted)
````

Para ver los parámetros disponibles o ver ayuda de las opciones hago:

````
slcli --help
slcli vs --help
slcli vs create --help
````

Por más información, la documentación oficial está en:

http://softlayer-python.readthedocs.io/en/latest/cli.html#usage-examples