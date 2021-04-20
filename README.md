# Dockerized SMIS server configuration and run

## Supported Intelliflash versions matrix

|  Docker version                      | SMI-S version 1.0             |
| ------------------------------------ | ----------------------------- |
|  => 19.03                            | Intelliflash version 3.11.0.4 |

## How to configure SMIS server before container run

SMIS docker container should have configuration yaml file, by default it's `config.yaml` and should be mapped into `/etc/cimserver/config.yaml` inside the simserver container

```yaml
#
# Proposal yaml for cimserver configuration
#
cimserver:
  properties: # map to describe cimserver start props
    enableAuthentication: true
    sslBackwardCompatibility: true
#    enableNamespaceAuthorization: true
#    logLevel: TRACE
#    traceComponents: XmlIO
#    traceLevel: 5
#    traceFacility: Log
  cert:
#    countryName: US # default value US
#    stateOrProvinceName: California # default value California
#    organizationName: # default value is 'DataDirect Networks, Inc'
#    organizationalUnitName: # default value 'OpenPegasus cimserver instance'
#    commonName: cimserver.IP # IP or FQDN of the Cimserver certificate
#    expiration: 365 # Certificate expiration in days, default valie is 365
#    privateKeyPath: <externalPrivateKeyPath>
#    certificatePath: <externalCertificatePath>
array:
  endpoint: 10.20.30.40
  user: admin
  password: dA==
  users:
    - user: user1
      password: dXNlcjE=
      permissions:
        - namespace: Intelliflash
          rights: "RW"

```

### Fields and values
Field name                           | Value   | Description
------------------------------------ | ------- | --------------------------------------------------------------------
cimserver.properties                 | map     |  OpenPegasus cimserver' initial properties should be described there. See cimserver admin's guide `pegasus/doc/Admin_Guide_Release.pdf`
array                                | struct  |  Describes confuguration for remote array
array.endpoint                       | string  |  IP/FQDN of the backend array
array.user                           | struct  |  Backend's connection user name
array.password                       | struct  |  Backend's connection password. Note: 'Should be encoded in BASE64. Example: `echo -n 'password' | base64`'.
array.users                          | array   |  Cimserver's users. Enables BASICAUTH for every remote request. Should be used when `enableAuthentication=true`
array.users.[n].user                 | string  |  Cimserver's user name
array.users.[n].password             | string  |  Cimserver's user password. Note: 'Should be encoded in BASE64. Example: `echo -n 'password' | base64`'.
array.users.[n].permissions          | string  |  Cimserver's user namespace permissions.
array.users.[n].permissions.namespace| string  |  Cimserver's user namespace name
array.users.[n].permissions.rights   | string  |  Cimserver's user namespace permissions. `R` is for READ authorization, `W` is for WRITE authorization
cimserver.cert                       | cert    |  Cimserver's certificate configuration struct
cimserver.cert.countryName           | cert    |  Self-signed certificate 'C' attribute. Default value is `US`.
cimserver.cert.stateOrProvinceName   | cert    |  Self-signed certificate 'ST' attribute. Default value is `California`.
cimserver.cert.organizationName      | cert    |  Self-signed certificate 'O' attribute. Default value is `DataDirect Networks, Inc`.
cimserver.cert.organizationalUnitName| cert    |  Self-signed certificate 'OU' attribute. Default value is `OpenPegasus cimserver instance`.
cimserver.cert.commonName            | cert    |  Self-signed certificate 'CN' attribute. If not defined, then IP address of the container will be used.
cimserver.cert.expiration            | cert    |  Self-signed certificate expiration in days. Default value `365`
cimserver.cert.privateKeyPath        | cert    |  External private key file path. `certificatePath` must be specified too. It's container path, not host.
cimserver.cert.certificatePath       | cert    |  External Certificate file path. `privateKeyPath`  must be specified too. It's container path, not host.

`Note:` If `privateKeyPath` and `certificatePath` specified then other `cert` options are omitted.

`Note:` SMIS configuration example placed at `https://github.com/DDNStorage/Intelliflash-smi-s-plugin/config/config.yaml`

## How to load SMIS container image

1. Clone repository
```bash
git clone https://github.com/DDNStorage/Intelliflash-smi-s-plugin /opt/Intelliflash-smi-s-plugin
```

2. Untar image
```bash
tar -zxvf /opt/Intelliflash-smi-s-plugin/bin/smis-server.tar.gz -C /opt/Intelliflash-smi-s-plugin/bin/
```

3. Load image
```bash
docker load -i /opt/Intelliflash-smi-s-plugin/bin/smis-server.tar
```

## How to start SMIS container

```bash
docker run -dit -p 5988:5988 -p 5989:5989 -v /opt/Intelliflash-smi-s-plugin/config:/etc/cimserver smis-server:1.0
```

## How to stop SMIS container
```bash
docker ps | grep -i -m 1 smis-server | awk '{print $1}' | xargs docker kill
```
