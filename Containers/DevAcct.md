# DevAcct Container

### For Image Build/Pull
-`docker build --load -t DevAcct1 .`


### To Run the container 

   - To Ru in Deteched Mode.
    `docker run -d -p 8000:8000 --name my-acct-container DevAcct`
 - To execute a migration command in running container.
  `docker exec my-acct-container python manage.py migrate`
