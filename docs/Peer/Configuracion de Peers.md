# Peers

En los peers está la configuración del peer y el msp (los certificados y claves para los distintos canales).

Un deploy con Ansible sería: el binario y la configuración. Las dependencias serían el ordering service y couchdb.

## ¿Que hay en los peers?

La configuración en /etc/hyperledger/fabric
El binario en /usr/local/bin/peer

peer node start

La configuración:

configtx.yaml  core.yaml  msp  orderer.yaml  tls

En tls
root@a8c07eac507c:/etc/hyperledger/fabric# ls tls
ca.crt  server.crt  server.key

En msp
root@a8c07eac507c:/etc/hyperledger/fabric# ls -ltR msp/
msp/:
total 20
drwxr-xr-x 2 1000 1000 4096 Nov 15 12:22 admincerts
drwxr-xr-x 2 1000 1000 4096 Nov 15 12:22 cacerts
drwxr-xr-x 2 1000 1000 4096 Nov 15 12:22 tlscacerts
drwxr-xr-x 2 1000 1000 4096 Nov 15 12:22 keystore
drwxr-xr-x 2 1000 1000 4096 Nov 15 12:22 signcerts

msp/admincerts:
total 4
-rw-rw-r-- 1 1000 1000 790 Nov 15 12:22 Admin@org1.example.com-cert.pem

msp/cacerts:
total 4
-rw-rw-r-- 1 1000 1000 847 Nov 15 12:22 ca.org1.example.com-cert.pem

msp/tlscacerts:
total 4
-rw-rw-r-- 1 1000 1000 855 Nov 15 12:22 tlsca.org1.example.com-cert.pem

msp/keystore:
total 4
-rwx------ 1 1000 1000 241 Nov 15 12:22 26cd95cf51def95d1ef20b9d905f7c9a86c2a890ff58649b919a2cb0fb040aeb_sk


## ¿Y CouchDB? 

No necesita correr en el peer, pero en el peer se configura donde corre.

Igual en la configuración dice donde hallarlo.

## ¿Y Kafka?

También se configura aquí donde se accede. Por defecto, no lo usa. En este caso va para el orderer.
