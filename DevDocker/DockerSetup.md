## DevDocker Docker Setup and Configuration 

### Steps to Install Docker]

- sudo apt-get update
- sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release software-properties-common
- sudo mkdir -p /etc/apt/keyrings
- curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
- echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
- sudo apt-get update
- sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
- Edit the /etc/groups file (use sudo), and add your username at the end of the line with docker (after the colon)
- Log out and log back in
- docker info
- docker run hello-world
- sudo usermod -aG docker newusername


## Secure Docker Deamon:

### Create Certificates
### Server Certificate Generation

Note
Replace all instances of $HOST in the following example with the DNS name of your Docker daemon's host.


- $ `openssl genrsa -aes256 -out ca-key.pem 4096`
Generating RSA private key, 4096 bit long modulus
Generating RSA private key, 4096 bit long modulus
..............................................................................++
........++
e is 65537 (0x10001)
Enter pass phrase for ca-key.pem:
Verifying - Enter pass phrase for ca-key.pem:

- $ `openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem`  
Enter pass phrase for ca-key.pem:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:Queensland
Locality Name (eg, city) []:Brisbane
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Docker Inc
Organizational Unit Name (eg, section) []:Sales
Common Name (e.g. server FQDN or YOUR name) []:$HOST
Email Address []:Sven@home.org.au

   Now that you have a CA, you can create a server key and certificate signing request (CSR). Make sure that "Common Name" matches the hostname you use to connect to Docker:

- $ `openssl genrsa -out server-key.pem 4096`
Generating RSA private key, 4096 bit long modulus
.....................................................................++
.................................................................................................++
e is 65537 (0x10001)

- $ `openssl req -subj "/CN=$HOST" -sha256 -new -key server-key.pem -out server.csr`
  Next, we're going to sign the public key with our CA:

Since TLS connections can be made through IP address as well as DNS name, the IP addresses need to be specified when creating the certificate. For example, to allow connections using 10.10.10.20 and 127.0.0.1: Here in sted of 10.10.10.20
you can provide private ip address of server and inplace of $HOST provide u r dns that points to this server via public ip.

- $ `echo subjectAltName = DNS:$HOST,IP:10.10.10.20,IP:127.0.0.1 >> extfile.cnf`
Set the Docker daemon key's extended usage attributes to be used only for server authentication:

- $ `echo extendedKeyUsage = serverAuth >> extfile.cnf`

Now, generate the signed certificate:

- $ `openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem \
  -CAcreateserial -out server-cert.pem -extfile extfile.cnf`

  Signature ok
subject=/CN=your.host.com
Getting CA Private Key
Enter pass phrase for ca-key.pem:

### Client Certificate Generation

For client authentication, create a client key and certificate signing request:

- $ `openssl genrsa -out key.pem 4096`
Generating RSA private key, 4096 bit long modulus
.........................................................++
................++
e is 65537 (0x10001)

- $ `openssl req -subj '/CN=client' -new -key key.pem -out client.csr`

To make the key suitable for client authentication, create a new extensions config file:

- $ `echo extendedKeyUsage = clientAuth > extfile-client.cnf`

Now, generate the signed certificate:

- $ `openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem \
  -CAcreateserial -out cert.pem -extfile extfile-client.cnf`
Signature ok
subject=/CN=client
Getting CA Private Key
Enter pass phrase for ca-key.pem:

After generating cert.pem and server-cert.pem you can safely remove the two certificate signing requests and extensions config files:

- $ `rm -v client.csr server.csr extfile.cnf extfile-client.cnf`

With a default umask of 022, your secret keys are world-readable and writable for you and your group.

To protect your keys from accidental damage, remove their write permissions. To make them only readable by you, change file modes as follows:

- $ `chmod -v 0400 ca-key.pem key.pem server-key.pem`

Certificates can be world-readable, but you might want to remove write access to prevent accidental damage:

- $ `chmod -v 0444 ca.pem server-cert.pem cert.pem`

After you complete these steps, ensure that the following five files are generated.
ca.pem
server-cert.pem
server-key.pem
cert.pem
key.pem

## Configure docker deamon for tls

Below you can find the procedure of configuring docker to listen to TCP 2376 by using the systemctl method. systemctl is the most popular tool to configure the way apps start and run in Linux. You can later mix this method with other options, such as configuring docker to start each time the Operating System starts. If you tried the first method (by using the daemon.json file) and it did not work, you need to delete the folder /etc/docker/ before you try the second method.

rocedure:
1. Use the `sudo nano /etc/systemd/system/docker.service.d/override.conf` command to open an override file for docker service in a text editor.
2. Add or modify the following lines, substituting your own machine static IP

[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2376 --tlsverify --tlscacert=<location of ca certificate> --tlscert=<location of server certificate> --tlskey=<location of server key>
e.g.:ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2376 --tlsverify --tlscacert=/home/nilkanth/ca2/ca.pem --tlscert=/home/nilkanth/ca2/server-cert.pem --tlskey=/home/nilkanth/ca2/server-key.pem

Important: Do not remove the first line that contains ExecStart=. It needs to remain with an empty value.

3. Once the file is saved, you can exit the systemctl tool by using Ctrl+X > Y/y > Enter.
4. Reload the systemctl configuration by using the following command
`sudo systemctl daemon-reload`

5. Restart the Docker service by using the following command
   `sudo systemctl restart docker.service`

6. To check whether the changes were successfully applied or that docker is listening on the configured port, use the following command.
   `sudo netstat -lntp | grep dockerd`

## Configure docker on client (Windows PC)

First Install docker desktop or equvelent docker ce version on windows machine without any Virtual machine or any
other virtual support. Then follow below steps.

1. Copy necessary 3 certificate files from server which you had create in above steps to your local folder using following command.
2. `c:dev\py\cert>Copy ca.pem, cert.pem & key.pem to this location`

3. Set below variables as Enviornment variables in windows using System Properties --> Environment Variables. Here add follwoing three valriables and set its values accordingly.

  `DOCKER_CERT_PATH=c:\dev\py\cert1\  (location where you store certificates from server)
DOCKER_HOST=tcp://$HOST:2376
DOCKER_TLS_VERIFY='1'`

4. Test the docker client using following two commands. If tls is configured correctly or not?
   `c:\dev\py\cert1>docker info`
   `c:\dev\py\cert1>docker ps`

5. To troubleshoot make sure port 2376 is allowed on server and client firewalls. and check using following tool.
`c:\dev\py\cert1>telnet $HOST 2376`

If telnet gets connected your docker should be working otherwise check firewall on server and client.

***



### Docker Command HandToolSet

#### To Delete all Running Container Forcefully
- docker rm -f $(docker ps -a -q)
- #### Delete all Images
- docker rmi -f $(docker images -q)
- #### Delete all Volumes which are not IIn Use
- docker stop $(docker ps -a -q)
docker volume rm $(docker volume ls -q)
