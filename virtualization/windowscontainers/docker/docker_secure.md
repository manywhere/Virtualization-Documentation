---
title: Connecting with the Docker daemon
description: Connecting with the Docker daemon
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-contianers
ms.service: windows-containers
ms.assetid: 2605ece3-5918-4898-bc6b-60ef09e44d9d
---

# Connecting with the Docker daemon

The Docker daemon on Windows accepts client and API connections locally through a Windows named piped and remotely through a TCP port. As a best practice, the Docker daemon should be accessd from a remote machine through a TCP port. Furthermore, when doing so, the connection should be secured with SSL. This document will detail connecting with the Docker daemon, and how to secure these connections on Windows.

For detailed information on securing the Docker daemon, see [Protect the Docker daemon socket on Docker.com](https://docs.docker.com/engine/security/https/).


## Docker daemon startup

### Named pipe

With nothing specified, the daemon will only accept local connections through the named pipe.

```none
docker daemon
```

Specifying the named pipe is equivalent to the last example.

```none
docker daemon -H npipe:// 
```

### TCP port

A TCP port can also be specified using the -H parameter. In this example port 2375 is exposed however is not secured. In this configuration no access control is configured, so anyone can deploy a container to the host over this port. This exposes the potential for malicious container execution. This configuration is not recommended on any configuration. 

> If a TCP port is specified, and the named pipe will also be used, the named pipe declaration must be made as seen in this example.

```
dockerd -H npipe:// -H 0.0.0.0:2375
```

The TCP port can be secured using SSL certificates.

```none
dockerd -H npipe:// -H 0.0.0.0:2376 --tlsverify --tlscacert=C:\ProgramData\Docker\certs.d\ca.pem --tlscert=C:\ProgramData\Docker\certs.d\server-cert.pem --tlskey=C:\ProgramData\Docker\certs.d\server-key.pem
```

## SSL Certificates

### Server SSL Certificates

Generate CA Private and Public keys:

	openssl genrsa -aes256 -out ./cert/ca-key.pem 4096
	openssl req -new -x509 -days 365 -key ./cert/ca-key.pem -sha256 -out ./cert/ca.pem

Create server key and signing request (CSR):

	openssl genrsa -out ./cert/server-key.pem 4096
	openssl req -subj "/CN=WIN-33A3HTEISE3" -sha256 -new -key ./cert/server-key.pem -out ./cert/server.csr

Sign the public key:

	echo subjectAltName = IP:10.0.0.13,IP:127.0.0.1 > ./cert/extfile.cnf
	openssl x509 -req -days 365 -sha256 -in ./cert/server.csr -CA ./cert/ca.pem -CAkey ./cert/ca-key.pem -CAcreateserial -out ./cert/server-cert.pem -extfile ./cert/extfile.cnf
    
### Client SSL Certificates

Create client key and signing request:

	openssl genrsa -out ./cert/key.pem 4096
	openssl req -subj "/CN=client" -new -key ./cert/key.pem -out ./cert/client.csr
	
Sign the public key:

	echo extendedKeyUsage = clientAuth > ./cert/extfile.cnf
	openssl x509 -req -days 365 -sha256 -in ./cert/client.csr -CA ./cert/ca.pem -CAkey ./cert/ca-key.pem -CAcreateserial -out ./cert/cert.pem -extfile ./cert/extfile.cnf


## Docker Daemon system

### Install the Docker daemon

```none
wget https://aka.ms/tp5/dockerd -OutFile $env:SystemRoot\system32\dockerd.exe
```

### Secure the Docker daemon

Copy the server SSL certificates to the ` C:\ProgramData\Docker\certs.d` folder on the container host.

## Docker client system

### Install the Docker client

```none
wget https://aka.ms/tp5/docker -OutFile $env:SystemRoot\system32\docker.exe
```

### Secure the Docker client

Create an environmental variable named `DOCKER_CERT_PATH` and give it a value of the directory where the client SSL certificates will be copied.
```none
$env: DOCKER_CERT_PATH = "c:\dokcercerts"
```
Copy the client SSL certificates into the `DOCKER_CERT_PATH` directory.
