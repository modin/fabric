# Crear CA para Fabric CA

## Motivación

Fabric CA sirve para generar identidades y darles permisos sobre Fabric, p.e. miembro que puede ejecutar transacciones, peer, orderer. Todos los componentes de Fabric deben estar identificados y autenticarse (permissioned blockchain).

Fabric CA centraliza la generación del material criptográfico necesario, en la forma de certificados x509 y claves privadas (usando ECDSA y SHA256).

Pero, ¿quién garantiza que Fabric CA es realmente quien dice ser? Fabric CA debe usar un certificado firmado por una entida certificadora real, como el Correo, Verisign, etc. Luego, puede firmar cosas y quienes vean esas firma van a poder validarla (o descubrir si no es auténtica).

## CA de prueba

Si bien es posible levantar Fabric CA con un certificado autofirmado, esto no es como funcionaría en la realidad. Para armar un laboratorio lo más cercano a la realidad (cada miembro de la red de negocios podría tener su propia Fabric CA) armarmos una 'CA', esto es, una entidad certificadora que firmará el certificado de los Fabric CA.

La estructura de directorios final es
```
root/
└── ca
    ├── certs
    │   └── ca-ec.cert.pem
    ├── crl
    ├── index.txt
    ├── index.txt.attr
    ├── index.txt.old
    ├── intermediate
    │   ├── certs
    │   │   ├── ca-chain.cert.pem
    │   │   └── intermediate-ec.cert.pem
    │   ├── crl
    │   ├── crlnumber
    │   ├── csr
    │   │   └── intermediate-ec.csr.pem
    │   ├── index.txt
    │   ├── newcerts
    │   ├── openssl.cnf
    │   ├── private
    │   │   └── intermediate-ec.key.pem
    │   └── serial
    ├── newcerts
    │   └── 1000.pem
    ├── openssl.cnf
    ├── private
    │   └── ca-ec.key.pem
    ├── serial
    └── serial.old
```

Por razones de comodidad, el directorio raíz donde ponemos la clave, certificate request y certificado x509 de la entidad intermedia (el Fabric CA), lo incluímos en el árbol de subdirectorios.

Luego, a nivel del filesystem

```
fabric-ca
├── docker-compose.yml
├── fabric-ca-server
│   ├── ca-cert.pem
│   ├── ca-chain.pem
│   ├── ca-key.pem
│   ├── fabric-ca-client-config.yaml
│   ├── fabric-ca-server-config.yaml
│   ├── fabric-ca-server.db
│   └── msp
│       ├── cacerts
│       │   └── localhost-7054.pem
│       ├── intermediatecerts
│       │   └── localhost-7054.pem
│       ├── keystore
│       │   ├── 47acefe2b430219b79529c9af55a9154cba28f4a53c3964fd1a81750d20090af_sk
│       │   └── a6a62a57967a51a7ffaf250a9dcdfc1a7f71bb8a300b0d399d0523b8ab0de8a2_sk
│       └── signcerts
│           └── cert.pem
├── intermediate-ec.cert.pem
└── intermediate-ec.key.pem
```

## Procedimiento


Actores: administradores de la CA, administradores de Fabric CA, clientes de Fabric CA

_Crear CA (lo hacen los admins de la CA)_

1 - Creo clave privada

`openssl ecparam -name prime256v1 -genkey -out private/ca-ec.key.pem`

2 - Creo certificado autofirmado

`openssl req -config openssl.cnf -key private/ca-ec.key.pem -new -x509 -days 7300 -sha256 -extensions v3_ca -out certs/ca-ec.cert.pem`

(Defaults y otros parámetros van en openssl.cnf)

_Crear clave y certificado para Fabric CA (lo hacen los admin de Fabric CA)_

![alt text](./Diagram1.png)

3 - Creo clave privada de entidad intermedia (cada Fabric CA deberá hacer esto)

`openssl ecparam -name prime256v1 -genkey -out intermediate/private/intermediate-ec.key.pem`

4 - Creo certificate request

`openssl req -config intermediate/openssl.cnf -new -sha256 -key intermediate/private/intermediate-ec.key.pem -out intermediate/csr/intermediate-ec.csr.pem`

5 - La CA (creada arriba, en pasos 1 y 2) firma y devuelve un certificado x509 para Fabric CA

`openssl ca -config openssl.cnf -extensions v3_intermediate_ca -days 3650 -notext -md sha256 -in intermediate/csr/intermediate-ec.csr.pem -out intermediate/certs/intermediate-ec.cert.pem -policy policy_loose`

_Crear configuración y levantar Fabric CA (lo hacen los admin de Fabric CA)_

6 - Creo configuración, con defaults (ver apéndice)
7 - Levando servicio (puedo clusterizarlo)

Usando docker-compose.yml
```
fabric-ca-server:
   image: hyperledger/fabric-ca:x86_64-1.0.4
   container_name: fabric-ca-server
   ports:
     - "7054:7054"
   environment:
     - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
   volumes:
     - "./fabric-ca-server:/etc/hyperledger/fabric-ca-server"
   command: sh -c 'fabric-ca-server start -b admin:adminpw'
```

_Enrolar identidad a Fabric (lo hacen los clientes de Fabric CA)_

8 - Creo configuración, con defaults (ver apéndice)
9 - Invoco fabric-ca-client (binario o contenedor)

`fabric-ca-client enroll -u http://admin:adminpw@localhost:7054`

_Asignar permisos a identidad (lo hacen los clientes de Fabric CA)_


## Pendientes

Permisos en directorios y claves privadas protegidas por password
Eliminación de usuarios por defecto (admin:adminpw)
Conexión segura al Fabric CA (conf TLS)
Clusterizar Fabric CA para HA