# ClearShot

## Introduction

Welcome to this CTF challenge, where you'll exploit CVE-2023-32315 in Openfire, a messaging and group chat server. This vulnerability allows an authentication bypass via a path traversal attack, granting unauthorized access to application files and enabling Remote Code Execution (RCE). Your goal is to exploit this flaw, gain access, and capture the flag.

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
git clone https://github.com/luzifer-docker/openfire

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

[

Provide an in-depth explanation of the steps it takes to complete the box from start to finish. Divide your walkthrough into the below sections and sub-sections and include images to guide the user through the exploitation. 

Please also include screenshots of any visual elements (like websites) that are part of the submission. Our review team is not only evaluating the technical path, but the realism and story of the box.

Show **all** specific commands using markdown's triple-backticks (```` ```bash ````) such that the reader can copy/paste them, and also show the commands' output through images or markdown code blocks (```` ``` ````). 

**A reader should be able to solve the box entirely by copying and pasting the commands you provide.**

]

# Enumeration

[Describe the steps that describe the box's enumeration. Typically, this includes a sub-heading for the Nmap scan, HTTP/web enumeration, etc.]

# Foothold

[Describe the steps for obtaining an initial foothold (shell/command execution) on the target.]

# Lateral Movement (optional)

[Describe the steps for lateral movement. This can include Docker breakouts / escape-to-host, etc.]

# Privilege Escalation

[Describe the steps to obtaining root/administrator privileges on the box.]
