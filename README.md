# ClearShot

## Introduction

Welcome to this CTF challenge, where you'll exploit CVE-2023-32315 in Openfire, a messaging and group chat server. This vulnerability allows an authentication bypass via a path traversal attack, granting unauthorized access to application files and enabling Remote Code Execution (RCE). Your goal is to exploit this flaw, gain access, and capture the flag.

## Skills Required
Basic Linux

Basic Network Enumeration

Running scripts

## Skills Learned
Identifying vulnerable services

XMPP Enumeration and Exploitation 

Remote Code Execution

## Info for HTB

### Access

Passwords:

| User  | Password                            |
| ----- | ----------------------------------- |
|openfire| AdminNotHerehtb#2024 |



## Key Processes

Openfire
The core process of the Openfire server, responsible for managing XMPP communications, user authentication, message routing, and group chat functionality.

Ports:

9090: HTTP port for the Openfire Admin Console.

9091: HTTPS port for the Openfire Admin Console.

5222: Client-to-Server communication port.

5223: Secure Client-to-Server communication port.

5269: Server-to-Server communication port.

5005: JVM Debugging port.




## Docker

This docker image is being used with some slight modification.
```
git clone https://github.com/luzifer-docker/openfire
```

DockerFile
```
FROM alpine

LABEL maintainer Knut Ahlers <knut@ahlers.me>

ENV OPENFIRE_VERSION=4_7_4

RUN set -ex \
 && apk --no-cache add \
      bash \
      ca-certificates \
      curl \
      openjdk11 \
 && mkdir -p /opt \
 && curl -sSfL "https://www.igniterealtime.org/downloadServlet?filename=openfire/openfire_${OPENFIRE_VERSION}.tar.gz" | \
      tar -xz -C /opt \
 && curl -sSfLo /usr/local/bin/dumb-init https://github.com/Yelp/dumb-init/releases/download/v1.2.1/dumb-init_1.2.1_amd64 \
 && chmod +x /usr/local/bin/dumb-init

ADD start.sh /usr/local/bin/start.sh

EXPOSE 9090 9091 5222 5223 5269
VOLUME ["/data"]

ENTRYPOINT ["/usr/local/bin/start.sh", "-remotedebug"]

```

start.sh 
```
#!/usr/local/bin/dumb-init /bin/bash
set -euo pipefail

# init configuration
[ -e "/data/security/keystore" ] || {
        mkdir -p /data/security
        mv /opt/openfire/resources/security/keystore /data/security/keystore
}

[ -d "/data/embedded-db" ] || { mkdir -p /data/embedded-db; }
[ -d "/data/conf" ] || { mv /opt/openfire/conf /data/conf; }

ln -sfn /data/security/keystore /opt/openfire/resources/security/keystore
ln -sfn /data/embedded-db /opt/openfire/embedded-db
rm -rf /opt/openfire/conf && ln -sfn /data/conf /opt/openfire/conf

# start openfire
/opt/openfire/bin/openfire start

# let openfire start
echo "Waiting for Openfire to start..."
count=0
while [ ! -e /opt/openfire/logs/stdoutt.log ]; do
        if [ $count -eq 60 ]; then
                echo "Error starting Openfire. Exiting"
                exit 1
        fi
        count=$((count + 1))
        sleep 1
done

# tail the log
tail -F /opt/openfire/logs/*.log
```


# Writeup

# Enumeration
## Nmap
We start things off by performing an nmap scan against the target IP address.
```
nmap <target-ip>
```
By performing the nmap scan, we get the following result.
![image](https://github.com/WhitewolfX01/Test/assets/126961828/2ec2a322-7757-4af0-9c62-5ca6316a4ed4)

# Foothold

[Describe the steps for obtaining an initial foothold (shell/command execution) on the target.]

# Lateral Movement (optional)

[Describe the steps for lateral movement. This can include Docker breakouts / escape-to-host, etc.]

# Privilege Escalation

[Describe the steps to obtaining root/administrator privileges on the box.]
