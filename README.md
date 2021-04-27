# Dockerized SMIS server configuration and run

## Supported Intelliflash versions matrix

|  Docker version                      | SMIS version 1.0             |
| ------------------------------------ | ----------------------------- |
|  => 19.03                            | Intelliflash version 3.11.0.4 |

`Note:` If you use earlier Intelliflash version with activated SMIS you should upgrade your array to use Dockerized SMIS server and replace the old SMIS provider with the new docker SMIS provider connection.

## How to configure SMIS server before container run

SMIS docker container should have configuration yaml file, by default it's `config.yaml` and should be mapped into `/etc/cimserver/config.yaml` inside the simserver container.

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
Field name                           | Value   | Mandatory | Description
------------------------------------ | ------- | --------- | --------------------------------------------------------------------
cimserver.properties                 | map     | true      | OpenPegasus cimserver' initial properties should be described there. See cimserver admin's guide `pegasus/doc/Admin_Guide_Release.pdf`
array                                | struct  | true      | Describes confuguration for remote array
array.endpoint                       | string  | true      | IP/FQDN of the backend array
array.user                           | struct  | true      | Backend's connection user name
array.password                       | struct  | true      | Backend's connection password. Note: 'Should be encoded in BASE64. Example: `echo -n 'password' | base64`'.
array.users                          | array   | false     | Cimserver's users. Enables BASICAUTH for every remote request. Should be used when `enableAuthentication=true`
array.users.[n].user                 | string  | false     | Cimserver's user name
array.users.[n].password             | string  | false     | Cimserver's user password. Note: 'Should be encoded in BASE64. Example: `echo -n 'password' | base64`'.
array.users.[n].permissions          | string  | false     | Cimserver's user namespace permissions.
array.users.[n].permissions.namespace| string  | false     | Cimserver's user namespace name
array.users.[n].permissions.rights   | string  | false     | Cimserver's user namespace permissions. `R` is for READ authorization, `W` is for WRITE authorization
cimserver.cert                       | cert    | false     | Cimserver's certificate configuration struct
cimserver.cert.countryName           | cert    | false     | Self-signed certificate 'C' attribute. Default value is `US`.
cimserver.cert.stateOrProvinceName   | cert    | false     | Self-signed certificate 'ST' attribute. Default value is `California`.
cimserver.cert.organizationName      | cert    | false     | Self-signed certificate 'O' attribute. Default value is `DataDirect Networks, Inc`.
cimserver.cert.organizationalUnitName| cert    | false     | Self-signed certificate 'OU' attribute. Default value is `OpenPegasus cimserver instance`.
cimserver.cert.commonName            | cert    | false     | Self-signed certificate 'CN' attribute. If not defined, then IP address of the container will be used.
cimserver.cert.expiration            | cert    | false     | Self-signed certificate expiration in days. Default value `365`
cimserver.cert.privateKeyPath        | cert    | false     | External private key file path. `certificatePath` must be specified too. It's container path, not host.
cimserver.cert.certificatePath       | cert    | false     | External Certificate file path. `privateKeyPath`  must be specified too. It's container path, not host.

`Note:` Provide any user's credentials when enableAuthentication=true option.

`Note:` If `privateKeyPath` and `certificatePath` specified then other `cert` options are omitted.

`Note:` SMIS configuration example placed at `https://github.com/DDNStorage/Intelliflash-smi-s-plugin/blob/master/config/config.yaml`.

## How to load SMIS container image

1. Clone repository
```bash
git clone https://github.com/DDNStorage/Intelliflash-smi-s-plugin /opt/Intelliflash-smi-s-plugin
```

2. Untar and load image
```bash
tar -zxvf /opt/Intelliflash-smi-s-plugin/bin/smis-server.tar.gz | xargs docker load -i
```

## How to modify SMIS configuration file

Before start SMIS provider, please prepare configuration file. E.g., in this manual we use /opt/Intelliflash-smi-s-plugin/config/config.yaml:
```bash
vim /opt/Intelliflash-smi-s-plugin/config/config.yaml
```

`Note:` For more information see "How to configure SMIS server before container run" chapter.

## How to start SMIS container

```bash
docker run -dit -p 5988:5988 -p 5989:5989 -v /opt/Intelliflash-smi-s-plugin/config:/etc/cimserver smis-server:1.0
```

If you want to use individual IP address for SMIS container you can create static IP (e.g., 10.10.10.10) on your docker host and run container with modified options:

```bash
docker run -dit -p 10.10.10.10:5988:5988 -p 10.10.10.10:5989:5989 -v /opt/Intelliflash-smi-s-plugin/config:/etc/cimserver smis-server:1.0
```

## How to connect to SMIS provider

To connect to the SMIS provider from SCVMM, use the IP/FQDN of the docker container (which defaults to docker host machine) and the ports that were specified in the docker run command.

## How to view logs

In case of any issues during start container or in container work you can see logs for SMIS docker container. Here you can find information about connection errors, password problems, etc:
```bash
docker ps | grep -i -m 1 smis-server | awk '{print $1}' | xargs docker logs
```

## How to stop SMIS container

```bash
docker ps | grep -i -m 1 smis-server | awk '{print $1}' | xargs docker kill
```
