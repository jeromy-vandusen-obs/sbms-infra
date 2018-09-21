# Spring Boot Microservices - Infrastructure

## Jenkins Job Configuration

1. On the main Jenkins dashboard page, click "New Item".
2. Enter "sbms-infra" in the "name" field, select "Pipeline", and click "OK".
3. Scroll down to the "Pipeline" section and in "Definition" select "Pipeline script from SCM".
4. In "SCM", select "Git".
5. In "Repository URL", enter "https://github.com/jeromy-vandusen-obs/sbms-infra.git".
6. Deselect "Lightweight checkout" and click "Save".

## To Do

* Automate the creation of the shared library in Jenkins.
* Automate the creation of TLS certificates for Docker servers and client.
* Automate the creation of TLS certificates for the microservices, and implement HTTPS on all of them.
* Automate the creation of servers for various environments.
* Figure out logging and automate that, too.
* Figure out how to automate the Jenkins setup more than it is now.

## Notes

Some of the manual processes I've done as part of setting up the environment that need to be automated are documented
here.

### Creating the MongoDB data directories

#### Pre-Production Environments

```bash
$ sudo mkdir -p /opt/sbms/dev/mongodb
$ sudo mkdir -p /opt/sbms/test/mongodb
$ sudo mkdir -p /opt/sbms/uat/mongodb
$ sudo chown -R root:docker /opt/sbms
$ sudo chmod -R 770 /opt/sbms
```

#### Production Environment

```bash
$ sudo mkdir -p /opt/sbms/mongodb
$ sudo chown -R root:docker /opt/sbms
$ sudo chmod -R 770 /opt/sbms
```

### Securing Docker

#### Docker Client (Jenkins/Build Host)

```bash
$ cd /opt/jenkins
$ sudo mkdir tls
$ cd tls
$ sudo openssl genrsa -aes256 -out ca-key.pem 4096
> $PASSWORD
> $PASSWORD
$ sudo openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem
> $PASSWORD
> CA
> Manitoba
> Winnipeg
> Online Business Systems
>
>
>
$ sudo openssl genrsa -out key.pem 4096
$ sudo openssl req -subj "/CN=jenkins" -new -key key.pem -out cert.csr
$ sudo vim cert.cnf
> extendedKeyUsage = clientAuth
$ sudo openssl x509 -req -days 365 -sha256 -in cert.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile cert.cnf
> $PASSWORD
$ sudo rm ca.srl cert.csr cert.cnf
$ sudo chmod 0400 ca-key.pem key.pem
$ sudo chmod 0444 ca.pem cert.pem
$ cd ..
$ sudo chmod 750 tls
$ sudo vim docker-compose.yml
# Add to "volumes":
> - "/opt/jenkins/tls/ca.pem:/root/.tls/ca.pem"
> - "/opt/jenkins/tls/key.pem:/root/.tls/key.pem"
> - "/opt/jenkins/tls/cert.pem:/root/.tls/cert.pem"
$ docker-compose up -d
```

#### Docker Server (Production and Pre-Production Hosts)

You will need the ca-key.pem and ca.pem files created in the previous section.

```bash
$ sudo openssl genrsa -out key.pem 4096
$ sudo openssl req -subj "/CN=docker-host" -sha256 -new -key key.pem -out cert.csr
$ sudo vim cert.cnf
# Note IP address. Be sure to use the correct one for each host. Also, list all appropraite hostnames in "DNS:" entries.
> subjectAltName = DNS:docker-host,IP:172.24.140.XX,IP:127.0.0.1
> extendedKeyUsage = serverAuth
$ sudo openssl x509 -req -days 365 -sha256 -in cert.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile cert.cnf
> $PASSWORD
$ sudo sudo rm ca-key.pem ca.srl cert.csr cert.cnf
$ sudo chmod 0400 key.pem
$ sudo chmod 0444 ca.pem cert.pem
$ sudo chown root:root ca.pem key.pem cert.pem
$ sudo mkdir /etc/docker/ssl
$ sudo mv ca.pem /etc/docker/ssl
$ sudo mv key.pem /etc/docker/ssl
$ sudo mv cert.pem /etc/docker/ssl
$ sudo vim /lib/systemd/system/docker.service
# Consider moving this to a json configuration file instead of modifying the systemd file.
> ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2376 --tlsverify --tlscacert=/etc/docker/ssl/ca.pem --tlscert=/etc/docker/ssl/cert.pem --tlskey=/etc/docker/ssl/key.pem
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

### Installing Chrome for Automated Testing

This works for Ubuntu. Currently, the Jenkins server uses Alpine Linux, and I'd like to keep it that way, so may need
to figure out how to install Chrome on Alpine, but keeping this documentation anyway in case I decide to make Jenkins
Ubuntu based instead.

```bash
$ echo "deb http://dl.google.com/linux/chrome/deb/ stable main" | sudo tee -a /etc/apt/sources.list.d/google-chrome.list
$ sudo apt-get update
$ sudo apt-get install google-chrome-stable
```
