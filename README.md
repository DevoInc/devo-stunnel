# Stunnel4 Dockerized image: built to run with devo-relay

The Devo relay, also referred to as the "in-house" relay, was designed to reside within a customer's secure network, receive events over an unencrypted channel, then forward them to the Devo cloud using SSL/TLS encryption. As a result, it is not equipped to receive inbound SSL connections.

However, your data environment may consist of several, separate secure networks, each containing data sources whose events you want to forward to Devo via a single relay. In order to send events over an encrypted channel to the Devo relay, we use a tunneling service, in this example, stunnel, Using another docker container to accept connections with SSL encryption, then forward the data to the [Devo-relay container](https://docs.devo.com/confluence/ndt/sending-data-to-devo/the-devo-in-house-relay/installing-the-devo-relay/install-on-docker).

## Needs to run Devo-Relay with Stunnel: (Modify in docker-devo-relay-stunnel.yml)

After installing Docker in your machine, simply clone this repo.

1. [Follow the instructions in the Relay In House Docker documentation](https://github.com/DevoInc/devo-relay/#needs-to-run-relay-modify-in-docker-devo-relayyml)
2. Change permissions to this files:

   - chmod 644 {project_path}/conf/stunnel/certs/server.crt
   - chmod 600 {project_path}/sconf/stunnel/certs/server.key
   - chmod 666 {project_path}/conf/logs/stunnel4/stunnel.log

3. You can check and download devo-stunnel image from [DockerHub](https://hub.docker.com/r/devoinc/devo-stunnel):

   - docker pull devoinc/devo-stunnel:5.44

## Start and Stop Devo-Relay with Stunnel

1. RUN: cd to {project_path} and execute the following command:

```
DCKHUB=devoinc/ docker-compose -f docker-devo-relay-stunnel.yml up -d
```

2. STOP: cd to {project_path} and execute the following command:

```
DCKHUB=devoinc/ docker-compose -f docker-devo-relay-stunnel.yml down
```

## Verify the installation

1. The docker containers are running.

```
$ docker ps
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                                  NAMES
947b0e942713        devoinc/devo-relay:1.1.6    "/opt/devo/docker-st…"   About an hour ago   Up About an hour    0.0.0.0:12999-13030->12999-13030/tcp   Devo-Relay
2fa63a34e2c8        devoinc/devo-stunnel:5.44   "stunnel"                About an hour ago   Up About an hour    0.0.0.0:6601-6650->6601-6650/tcp       Devo-Stunnel
```

2. Connect to Devo-Stunnel Container

```
$ docker exec -it Devo-Stunnel bash
[root @ stunnel:/opt/devo] $

```

3. Send test events

- without Stunnel, sending directly to the relay container.

```
$ for i in $( seq 100 ) ; do echo "<14>$( date --iso-8601=seconds ) docker-relay test.keep.free: Docker message to Devo-Relay directly port 13000 - test event # $i" | nc -N Devo-Relay 13000 ; done

```

- Stunnel.

```
$ for i in $( seq 100 ) ; do echo "<14>$( date --iso-8601=seconds ) docker-stunnel test.keep.free: Docker message with stunnel to Relay - test event # $i"|openssl s_client -connect localhost:6615 -key /etc/pki/stunnel/server.key -cert /etc/pki/stunnel/server.crt; done
```

4. View events in test.keep.free table.

## Customize Stunnel configuration.

The default stunnel conf file is located at {project_path}/conf/stunnel/stunnel.conf.
This basic configuration contains self-signed certificates that must be exchanged for your own certificates.

## Troubleshooting

1. Devo-Stunnel doesnt start.

The stunnel.log log file was regenerated and we have to change permissions again.

```
$ docker ps
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                                  NAMES
b88f954c13ad        devoinc/devo-relay:1.1.6    "/opt/devo/docker-st…"   5 seconds ago       Up 3 seconds        0.0.0.0:12999-13030->12999-13030/tcp   Devo-Relay
22f45048bc93        devoinc/devo-stunnel:5.44   "stunnel"                5 seconds ago       Up 3 seconds        0.0.0.0:6601-6650->6601-6650/tcp       Devo-Stunnel
```

And then start and stop Devo-Relay with Stunnel

## Author

© Devo Inc. 2020

## Contact Us
You can contact with us at support@devo.com.

## License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details

#### Enjoy it
