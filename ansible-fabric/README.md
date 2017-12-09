# Crear material criptografico

En esta version se hace con cryptogen, la configuracion esta en ./conf y los certificados y claves estaran en ./crypto. En el futuro debería ser una tarea ejecutada en los peers, o en la red de los peers, usando fabric-ca-client u openssl.

El comando:

cryptogen generate --config=./conf/crypto-config.yaml --output=./crypto

# Creacion del Genesis block (configuracion de la red)

Se basa en un profile definido en configtx.yaml. Hay que poner atencion al path, ya que la configuracion la busca en el path apuntado por la variable FABRIC_CFG_PATH y en el archivo configtx.yaml da el path a los certificados y claves.

Asumiento que conf y crypto están al mismo nivel, hago:

export FABRIC_CFG_PATH=$PWD/conf

configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./crypto/genesis-block

Para ver el contenido del bloque generado:

configtxgen -profile TwoOrgsOrdererGenesis -inspectBlock ./crypto/genesis-block
