 DevDocker Inintial Steps

 ### Add a Mew Iser amd configure it for ssh

 sudo adduser newusername
sudo usermod -aG sudo newusername
mkdir ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

echo "Insert Public Key here" >> ~/.ssh/authorized_keys


#### allow ssh server to forward a port via ssh
sudo nano /etc/ssh/sshd_config

#### set AllowTcpForwarding yes in above file and save and restart ssh server


### ssh based key generation and configuration process

#### Create a key per user. for which login to user and in /home/user/.ssh folder enter following command

$ ssh-keygen -t rsa -b 4096 -C "rudra@gmail.com"   # here use email address

#### from windows orinot use same folder as u ssh in to server to enter below command which will copy files from server to local client
c:\> scp rudra@13.201.71.254:/home/rudra/.ssh/abkey_rsa.pub .

#### In your Ubuntu server: You need to add your public key to the ~/.ssh/authorized_keys file. You can do this by logging into the server and running the following command:
$ echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQ...' >> ~/.ssh/authorized_keys  

#### eplace 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQ...' with the contents of your public key file. You can get these contents by opening the file in a text editor on your Windows machine.
$ chmod 600 ~/.ssh/authorized_keys



