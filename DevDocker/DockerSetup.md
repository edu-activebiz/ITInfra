## DevDocker Docker Setup and Configuration 
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

Steps to install:
+ sudo apt-get update
+ sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release software-properties-common
+ sudo mkdir -p /etc/apt/keyrings
+ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
+ echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
+ sudo apt-get update
+ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
- Edit the /etc/groups file (use sudo), and add your username at the end of the line with docker (after the colon)
- Log out and log back in
+ docker info
+ docker run hello-world
+sudo usermod -aG docker newusername

