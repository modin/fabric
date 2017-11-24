# Tutorial de uso de Fabric CA

Fabric CA server se usa para generar ECerts y TCerts. Precisa configuración, una clave privada que se genera así:

`openssl ecparam -name prime256v1 -genkey -out intermediate/private/intermediate-ec.key.pem`

Con esa clave, armo un CSR que envío a una CA para que me lo firme:

`openssl req -config intermediate/openssl.cnf -new -sha256 -key intermediate/private/intermediate-ec.key.pem -out intermediate/csr/intermediate-ec.csr.pem`

Cuando tengo las tres cosas, la clave, el certificado (firmado), y la ca chain, puedo proceder a instalar Fabric CA server. 

El server tiene en la configuración p.e. la base de datos, si usa TLS o no, si usa LDAP, las afiliaciones, etc.

## Instalación de Fabric CA con certificado de terceros

Hay que tener creado un directorio donde guardar la configuración.
`mkdir fabric-ca-server`

Ahí tengo que poner la configuracion del server (`fabric-ca-server-config.yaml`), la configuracion del client (`fabric-ca-client-config.yaml`) y la clave, certificado y ca chain (`ca-cert.pem,ca-chain.pem,ca-key.pem`)

Para levantar el servicio uso docker-compose:

```
fabric-ca-server:
   image: hyperledger/fabric-ca:x86_64-1.0.4  <--- ojo con la versión
   container_name: fabric-ca-server
   ports:
     - "7054:7054"
   environment:
     - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
   volumes:
     - "./fabric-ca-server:/etc/hyperledger/fabric-ca-server"
   command: sh -c 'fabric-ca-server start -b admin:adminpw'
```


## Enrollment de un usuario con "Fabric CA client"

La configuracion del CSR y de los atributos del cliente van en el archivo: `fabric-ca-client-config.yaml`

```
docker exec -ti fabric-ca "fabric-ca-client enroll -u http://admin:adminpw@localhost:7054" 

export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
fabric-ca-client enroll -u http://admin:adminpw@localhost:7054
```

Genera un Ecert correspondiente a la clave privada y CA chain en el "Fabric CA client msp dir"

## Registrar una nueva identidad

3 chequeos
  - si el invoker tiene en el atributo hf.Registrar.Roles peer,app,user puede registrar peers, apps y users pero no orderers

  - si el invoker tiene afiliacion a.b puede registrar identidad con afiliacion a.b.c pero no a.c (root affiliation es .)

  - si el invoker tiene en hf.Registrar.Attributes "a.b.*" puede registrar lo que sea que comience con "a.b."
  - si el invoker tiene en hf.Registrar "a.b.*,x.y.z", puede registrar "a.b.c,x.y.z" 

```
export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
fabric-ca-client register --id.name admin2 --id.affiliation org1.department1 --id.attrs 'hf.Revoker=true,admin=true:ecert'

fabric-ca-client register -d --id.name admin2 --id.affiliation org1.department1 --id.attrs '"hf.Registrar.Roles=peer,user",hf.Revoker=true'

fabric-ca-client register -d --id.name admin2 --id.affiliation org1.department1 --id.attrs '"hf.Registrar.Roles=peer,user"' --id.attrs hf.Revoker=true
```

_Ejemplos con valores por defecto:_

```
id:
  name:
  type: user
  affiliation: org1.department1
  maxenrollments: -1
  attributes:
    - name: hf.Revoker
      value: true
    - name: anotherAttrName
      value: anotherAttrValue
```

The following command would then register a new identity with an enrollment id of “admin3” which it takes from the command line, and the remainder is taken from the configuration file including the identity type: “user”, affiliation: “org1.department1”, and two attributes: “hf.Revoker” and “anotherAttrName”.

```
export FABRIC_CA_CLIENT_HOME=$HOME/fabric-ca/clients/admin
fabric-ca-client register --id.name admin3
```

Si hay múltiples CAs

`fabric-ca-client enroll -u http://admin:adminpw@localhost:7054 --caname <caname>`


## HSM

Fabric CA puede guardar claves privadas en HSM vía PKCS11 APIs (parte BCCSP block chain crypto provider de la configuracion).

Hay ejemplo con softhsm2.
