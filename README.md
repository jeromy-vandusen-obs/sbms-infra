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
