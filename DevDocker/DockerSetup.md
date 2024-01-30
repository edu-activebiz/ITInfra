## DevDocker Docker Setup and Configuration 

### Ste[s tp Install Docker]

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


### Secure Docker Deamon:

To enable and enforce TLS for your Docker daemon, you need to generate certificates and keys, and then configure Docker to use them. Here are the steps:

1. **Generate CA Private and Public Keys:**

   First, you need to generate a private key for your Certificate Authority (CA). You can do this with the `openssl` command:

   ```bash
   openssl genrsa -aes256 -out ca-key.pem 4096
   ```

   Then, generate a public key from the private key:

   ```bash
   openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem
   ```

   When running this command, you'll be asked for various pieces of information. These can be left blank.

2. **Generate Server Keys:**

   Next, create a server key and certificate signing request (CSR) for the Docker daemon:

   ```bash
   openssl genrsa -out server-key.pem 4096
   openssl req -subj "/CN=<your-hostname>" -sha256 -new -key server-key.pem -out server.csr
   ```

   Replace `<your-hostname>` with the hostname of your Docker daemon.

   Then, create an extensions file `server-extfile.cnf` for the server:

   ```bash
   echo subjectAltName = DNS:<your-hostname>,IP:<your-ip>,IP:127.0.0.1 > server-extfile.cnf
   ```

   Replace `<your-hostname>` with the hostname of your Docker daemon, and `<your-ip>` with its IP address.

   Now, sign the public key:

   ```bash
   openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile server-extfile.cnf
   ```

3. **Generate Client Keys:**

   Similar to the server keys, create a client key and CSR:

   ```bash
   openssl genrsa -out key.pem 4096
   openssl req -subj '/CN=client' -new -key key.pem -out client.csr
   ```

   Then, create an extensions file `client-extfile.cnf` for the client:

   ```bash
   echo extendedKeyUsage = clientAuth > client-extfile.cnf
   ```

   Now, sign the public key:

   ```bash
   openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile client-extfile.cnf
   ```

4. **Secure the Keys:**

   Set restrictive permissions on the keys:

   ```bash
   chmod -v 0400 ca-key.pem key.pem server-key.pem
   chmod -v 0444 ca.pem server-cert.pem cert.pem
   ```

5. **Configure Docker Daemon:**

   Finally, configure Docker to use the certificates:

   ```bash
   dockerd --tlsverify --tlscacert=ca.pem --tlscert=server-cert.pem --tlskey=server-key.pem -H=0.0.0.0:2376
   ```

   This command starts Docker daemon with TLS enabled and configured to use the generated certificates and keys.

Remember to replace the placeholders in the commands with your actual values. Also, these commands should be run on the Docker daemon host.



*************************************
*************************************
***********************************


Yes, both the server and client certificates can be generated on the server. After generating, you can move the client-related certificates (client-cert.pem and client-key.pem) to the client machine.

Here are the steps to move the client certificates to a Windows client machine:

1. **Generate Certificates:** Follow the steps in the previous response to generate the server and client certificates on the server.

2. **Transfer Certificates:** You can use a secure method like SCP (Secure Copy) to transfer the client certificates from the server to the client machine. If you have an SCP client installed on your Windows machine, you can use a command like this:

   ```bash
   scp user@your-server:/path/to/cert.pem /local/path
   scp user@your-server:/path/to/key.pem /local/path
   ```

   Replace `user@your-server` with your username and server address, `/path/to/cert.pem` and `/path/to/key.pem` with the paths to the client certificates on the server, and `/local/path` with the path where you want to store the certificates on your Windows machine.

3. **Configure Docker Client:** On the Windows machine, set the `DOCKER_HOST`, `DOCKER_TLS_VERIFY`, and `DOCKER_CERT_PATH` environment variables to configure the Docker client to use TLS:

   ```powershell
   $env:DOCKER_HOST = 'tcp://your-server:2376'
   $env:DOCKER_TLS_VERIFY = '1'
   $env:DOCKER_CERT_PATH = 'C:\path\to\certs'
   ```

   Replace `your-server` with your server address and `C:\path\to\certs` with the path to the directory containing the client certificates.

Remember, the client certificates contain sensitive information. Make sure to transfer them securely and store them in a secure location on the client machine.

*************************************
***********************************
************************************


### Docker Command HandToolSet

#### To Delete all Running Container Forcefully
- docker rm -f $(docker ps -a -q)
- #### Delete all Images
- docker rmi -f $(docker images -q)
- #### Delete all Volumes which are not IIn Use
- docker stop $(docker ps -a -q)
docker volume rm $(docker volume ls -q)
