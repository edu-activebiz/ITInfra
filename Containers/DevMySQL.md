# DevMySQL Container

### For Image Build/Pull
-`docker pull mysql:latest`


### To Run the container 

    Using defaukt username as root and password as x.123456789. This container will lsten on port 3306 which is  a default port and default username as root listening on local ip.

-`docker run --name my-mysql-container -p 3306:3306 -e YSQL_ROOT_PASSWORD=x.123456789 -d mysql:latest`

***