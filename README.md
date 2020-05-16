# Docker Swarm Demo
- Swarm is used for orchestration of docker containers. This is similar to Kubernetes which is more popular.
- In order to try Swarm we will need multiple nodes(machines) where on each service we will run different services. This is not possible locally instead we will use Docker playground at https://labs.play-with-docker.com/, using which we can create multiple nodes and orchestrate containers. Create two nodes and run spring app, mysql app in each of the nodes
- `Node-1$ docker swarm init`

        => Error response from daemon: could not choose an IP address to advertise since this system has multiple addresses on different interfaces (192.168.0.8 on eth0 and 172.18.0.82 on eth1) - specify one with --advertise-addr
        => Since these nodes are created dynamically we will not have a specific IP address. So pick one from above and advertise it
- `Node-1$ docker swarm init --advertise-addr 192.168.0.8`

        - You will see something like below,
        - **"docker swarm join --token SWMTKN-1-387uh4jqglvcw20pdomzn7syucm5u4dqabn931fg7wrly5qxrw-f3rkiiwt1rew8sjrskn4tbzt1 192.168.0.8:2377"**
        - Copy it and paste it in Node-2
- `Node-2$ docker swarm join --token SWMTKN-1-387uh4jqglvcw20pdomzn7syucm5u4dqabn931fg7wrly5qxrw-f3rkiiwt1rew8sjrskn4tbzt1 192.168.0.8:2377`

        - This node joined a swarm as a worker.
        - Node-1 is the leader and Node-2 is the worker. Only a Leader(Node-1) can see all nodes. Go to Node-1 and try `ls`
- `Node-1$ docker node ls`
- Similar to creating network we also need to create something called "overlay" which is same as network but in this case it is across machines(nodes) where network is within a machine
  
  - `Leader-Node$ docker network create --driver overlay spring_mysql_ntw`
  - `Leader-Node$ docker network ls`  
       - should see "spring_mysql_ntw"
       - worker nodes cannot run this command and similar many other commands that leader can run
  - Let's start services(within docker we call container & within swarm we call services)
  - `Leader-Node$ docker service create -d --network spring_mysql_ntw -e MYSQL_ROOT_PASSWORD=dev -e MYSQL_DATABASE=company --name database amarnath510/mysql-5.6-docker-image`
    - => If up you will see a hash displayed like "pwqvil9bjxafrj7e6bzdnkf7r"
  - `Leader-Node$ docker service ls`
    - (similar to "docker container ps" but as said before in Swarm context we call services not container)

    - NOTE: Sometimes apps take time to download and run. So wait like 10-15 seconds depending on your app start-up time and try again "docker service ls" 
    ```
    ID                  NAME                MODE                REPLICAS            IMAGE                                       PORTS
    pwqvil9bjxaf        database            replicated          1/1                 amarnath510/mysql-5.6-docker-image:latest   
    ```
    => You will see REPLICAS column which should have "1/1" => 1-instance-up-of-required-instances-which-is-1
    => Swarm launches a container but it might not be on the node we have ran the command instead it can be on different node
  - `Leader-Node$ docker container ls` 
    - If you container listed then the container is started in this node itself.
  - Now run the another service (same from Leader command, we should not worry about nodes in which services are up. Swarm will take care of it .. that is the whole point of orchestration tools. Docker tried to run services in different nodes, it balances the load)
  * `Leader-Node$ docker service create -d --network spring_mysql_ntw -p 80:8200 --name spring-boot-app amarnath510/spring-boot-mysql-app`
  * `Leader-Node$ docker service ls` 
  
    - (wait for sometime like more than normal as above command needs to download and run the service. If it is not up even after some time limit then something is broken)
  * `docker service ls`
  
  ```
    ID                  NAME                MODE                REPLICAS            IMAGE                                       PORTS
    pwqvil9bjxaf        database            replicated          1/1                 amarnath510/mysql-5.6-docker-image:latest   
    frsjejc8lljf        spring-boot-app     replicated          1/1                 amarnath510/spring-boot-mysql-app:latest    *:80->8200/tcp
    ```
  * To **check logs**,
  
  - `Leader-Node$ docker service logs -f database`
     - From leader node we can look at logs of any services which even might be running on different nodes
  * At the top you can see a port been opened (80, the one we mapped for 8200 of spring-boot-app). If we click on it, a url opens something like "http://ip172-18-0-82-bqvdc4tim9m000a8uafg-80.direct.labs.play-with-docker.com/"
  * Try from browser by appending out API path, 
    - http://ip172-18-0-82-bqvdc4tim9m000a8uafg-80.direct.labs.play-with-docker.com/company/employees
    - [] // should return empty array as there are no records in database yet
  * Open Postman, **POST:** http://ip172-18-0-82-bqvdc4tim9m000a8uafg-80.direct.labs.play-with-docker.com/company/employees/all
    with RAW-JSON data,
    ```
    [
      {
        "firstName": "amarnath1",
        "lastName": "chandana",
        "email": "amarnath@docker.com"
      },
      {
        "firstName": "sushma1",
        "lastName": "chandana",
        "email": "sushma@docker.com"
      }
    ]
    ```
  * **GET**, http://ip172-18-0-82-bqvdc4tim9m000a8uafg-80.direct.labs.play-with-docker.com/company/employees
    => Should see above records 



# Docker Swam Demo using docker-compose

### TODO
