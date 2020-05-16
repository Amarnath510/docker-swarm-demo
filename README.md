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
  * To **check logs**:
    - `Leader-Node$ docker service logs -f database`
    - From leader node we can look at logs of any services which even might be running on different nodes
    - At the top you can see a port been opened (80, the one we mapped for 8200 of spring-boot-app). If we click on it, a url opens something like "http://ip172-18-0-82-bqvdc4tim9m000a8uafg-80.direct.labs.play-with-docker.com/"
    - Try from browser by appending out API path, 
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


# Docker Visualization

- We can visualize how Swarm is managing containers and nodes. Use [docker-swarm-visualizer](https://github.com/dockersamples/docker-swarm-visualizer)
- Go to your playground where already two services are running (spring & mysql). Run the visualizer too as,

  `docker run -it -d -p 5000:8080 -v /var/run/docker.sock:/var/run/docker.sock dockersamples/visualizer`
  (You can run on any port we just used 5000)
- Now you will see a new port being exposed in the Docker as below,
  
  <img width="1284" alt="Screenshot 2020-05-16 at 11 13 31 PM" src="https://user-images.githubusercontent.com/4599623/82126520-ef2b4600-97ca-11ea-9165-58e0496a7169.png">


- Now click on the `5000` port link at the top to see below beautiful visualizer where we see what are the nodes, names(worker/manager), which service is running on this node etc.

<img width="995" alt="Screenshot 2020-05-16 at 11 15 38 PM" src="https://user-images.githubusercontent.com/4599623/82126553-2c8fd380-97cb-11ea-8b1e-a2778406e5a1.png">

- You can do `docker container ls` and stop any container by `docker stop <con-id>` and immediately switch to visualier tab to see the beauty of visualization. You can try stopping spring-app and see how Swarm will restart it.


# Docker Swam Demo using docker-compose
- Previous we have created leader-worker and then ran services manually. Now let's do it via "docker stack" which is similar to docker-compose
  - Login to Docker playground
  - Click on "settings icon" on left and select 1-manager(leader) and 1-worker.(It should be enough for our case else choose other options)
  - Let's copy our compose file at https://github.com/Amarnath510/spring-boot-mysql-compose/blob/master/docker-compose.yml to manager node
  - Go to manager node
  - `manager> vi docker-compose.yml`
      - => copy docker-compose.yml content to above file and save it (:wq)
    `manager> docker stack deploy -c docker-compose.yml spring-mysql-stack` (give a name to this stack)
    `manager> docker service ls`
    `manager> docker service logs -f <service-name-which-you-get-from-"docker service ls">`
  - Also run docker visualization to check the nodes and the services running in them
 
