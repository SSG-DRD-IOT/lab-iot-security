# TLS Configuration

![Image of Security Lab Slide](./images/slide.png)

## Objectives

To create a self-signed certificate authority and a certificate that we will use to enable encrypted communication on the edge network. This includes secure MQTT & HTTP connection with TLS.

TLS (Transport Layer Security) provides a secure communication channel between a client and server

In this lab we will setup the required certificates to use for authentication and configuration for MQTT-TLS and HTTP-TLS connections. MQTT uses port 8883 for secure connection. We will change the Mosquitto configuration file to enable secure communication with the MQTT broker.

## Generate Server Certificate Authority, Certificate and Key

First, we will need to generate a server certificate authority, certificate and key. These will be used to authenticate and encrypt the MQTT sensor data.

**Run these commands on the gateway console.**

It is important to note that while generating these, the **“Common Name”** parameter in step 1 above and step 3 should specify the server address, **“localhost”**. Otherwise it will not authenticate correctly.

``` bash
openssl req -new -x509 -days 365 -extensions v3_ca -keyout ca.key -out ca.crt
openssl genrsa -out server.key 2048
openssl req -out server.csr -key server.key -new
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365
```

### Move the CA, Certificate and Key files to the mosquitto configuration directory

Now move the CA, Certificate and Key file to the MQTT broker's configuration directories

Note: If the directories **ca_certificates** and **certs** are not there manually create them

```bash
mkdir -p /etc/mosquitto/ca_certificates
mkdir -p /etc/mosquitto/certs
mv ca.crt ca.key ca.srl /etc/mosquitto/ca_certificates
mv server.crt server.csr server.key /etc/mosquitto/certs
```

### Configure Mosquitto to use MQTT over TLS

Add the below changes to the **end of the mosquitto.conf file**

```bash
    vim /etc/mosquitto/mosquitto.conf
```

```bash
# Place your local configuration in /etc/mosquitto/conf.d/
bind_address 127.0.0.1
port 8883
cafile /etc/mosquitto/ca_certificates/ca.crt
certfile /etc/mosquitto/certs/server.crt
keyfile /etc/mosquitto/certs/server.key
tls_version tlsv1
```

To paste text into VIM using your mouse, you may need to use the paste mode.

:set mouse=v
:set paste

Then type "i" to enter insert mode and click the middle mouse button (usually) to paste


Make sure to stop and start mosquitto service after this change

```bash
systemctl stop mosquitto
systemctl restart mosquitto
```

Check to make sure mosquitto service is running properly with below command. The status should be "active (running)"

```bash
systemctl status mosquitto
```
