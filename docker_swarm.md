# How to docker swarm


`docker swarm init`  

`docker node ls`  
```
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
wun78h4i57us4ieeewqg3bhy3 *   ubuntu18            Ready               Active              Leader              19.03.9
```

### docker service

`docker service create alpine ping 8.8.8.8`  
```
imm9rc9lbu93xia3jongucmaz
overall progress: 1 out of 1 tasks 
1/1: running   [==================================================>] 
verify: Service converged 
```

`docker service ls`  
```
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
imm9rc9lbu93        elated_mcnulty      replicated          1/1                 alpine:latest       
```
`docker service ps elated_mcnulty`  
```
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
3rlf2z5s8w9n        elated_mcnulty.1    alpine:latest       ubuntu18            Running             Running 2 minutes ago
```

`docker service update imm9rc9lbu93 --replicas 3`  
```
imm9rc9lbu93
overall progress: 3 out of 3 tasks 
1/3: running   [==================================================>] 
2/3: running   [==================================================>] 
3/3: running   [==================================================>] 
verify: Service converged 
```

`docker service ls`  
```
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
imm9rc9lbu93        elated_mcnulty      replicated          3/3                 alpine:latest       
```

`docker service ps elated_mcnulty`  
```
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
3rlf2z5s8w9n        elated_mcnulty.1    alpine:latest       ubuntu18            Running             Running 7 minutes ago                            
i81s5b6qfjpe        elated_mcnulty.2    alpine:latest       ubuntu18            Running             Running about a minute ago                       
tchphqcn1654        elated_mcnulty.3    alpine:latest       ubuntu18            Running             Running about a minute ago                       
```
`docker container rm -f elated_mcnulty.1.3rlf2z5s8w9nmg263noamh8ap`  
`docker service ls`  
```
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
imm9rc9lbu93        elated_mcnulty      replicated          2/3                 alpine:latest       
```
`docker service ls`  
```
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
imm9rc9lbu93        elated_mcnulty      replicated          3/3                 alpine:latest       
```
`docker service ps elated_mcnulty`  
```
ID                  NAME                   IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR                         PORTS
cyocyxfjjik1        elated_mcnulty.1       alpine:latest       ubuntu18            Running             Running about a minute ago                                 
3rlf2z5s8w9n         \_ elated_mcnulty.1   alpine:latest       ubuntu18            Shutdown            Failed about a minute ago    "task: non-zero exit (137)"   
i81s5b6qfjpe        elated_mcnulty.2       alpine:latest       ubuntu18            Running             Running 13 minutes ago                                     
tchphqcn1654        elated_mcnulty.3       alpine:latest       ubuntu18            Running             Running 13 minutes ago          
```                         


### docker service create and update

### docker-machine  
`docker-machine`  
`docker-machine create node1`  
`docker-machine create node2`  
`docker-machine create node3`  

laczymy sie do kazdej maszyny po ssh i instalujemy dockera.  
`docker-machine ssh node1`  
`curl -fsSL https://get.docker.com -o get-docker.sh`  
`sh get-docker.sh`  

tworzymy cluster sworm'a
1. iicjujemy managera:  
docker@node1:~$ `docker swarm init --advertise-addr 192.168.99.100`  
Swarm initialized: current node (ysg01fjolcszgkjs03z0q7v3b) is now a manager.

To add a worker to this swarm, run the following command:

    `docker swarm join --token SWMTKN-1-099a06pscjsl00sbuibo0gjb8weyez3671qkg5gp18h0d7tniu-ciyekj0ayr415htr1gsllja4c 192.168.99.100:2377`  

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.


2. podlaczamy node2 jako worker:

docker@node2:~$ `docker swarm join --token SWMTKN-1-099a06pscjsl00sbuibo0gjb8weyez3671qkg5gp18h0d7tniu-ciyekj0ayr415htr1gsllja4c 192.168.99.100:2377`  
docker@node3:~$ `docker swarm join --token SWMTKN-1-099a06pscjsl00sbuibo0gjb8weyez3671qkg5gp18h0d7tniu-ciyekj0ayr415htr1gsllja4c 192.168.99.100:2377`  



3. wracamy na node1:  
docker@node1:~$ `docker node ls`  
```                                                                                        
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
ysg01fjolcszgkjs03z0q7v3b *   node1               Ready               Active              Leader              19.03.5
rr94g4izzlfrw5d66fi6rmqca     node2               Ready               Active                                  19.03.5
yy3jwa2mr9txwpy264agguz5m     node3               Ready               Active                                  19.03.5
```
upgradujemy node2 do roli managera.
`docker node update --role manager node2`  
```
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
ysg01fjolcszgkjs03z0q7v3b *   node1               Ready               Active              Leader              19.03.5
rr94g4izzlfrw5d66fi6rmqca     node2               Ready               Active              Reachable           19.03.5
yy3jwa2mr9txwpy264agguz5m     node3               Ready               Active                                  19.03.5
```

generujemy na node1 token dla node3 dla roli manager:
docker@node1:~$ `docker swarm join-token manager`     
```                                                                                         
To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-099a06pscjsl00sbuibo0gjb8weyez3671qkg5gp18h0d7tniu-4tk0ef0gp7gy4ljqtyedxxdyy 192.168.99.100:2377
```
4. Podlaczamy node3 do clustra jako kolejny manager.
docker@node3:~$ `docker swarm join --token SWMTKN-1-099a06pscjsl00sbuibo0gjb8weyez3671qkg5gp18h0d7tniu-4tk0ef0gp7gy4ljqtyedxxdyy 192.168.99.100:2377`  
This node joined a swarm as a manager.

wracamy na node1 i tworzymy serwis:
docker@node1:~$ `docker service create --replicas 3 alpine ping 8.8.8.8` 
```                                                                      
ex7l5v6e2pqlgbg92idxechza
overall progress: 3 out of 3 tasks 
1/3: running   [==================================================>] 
2/3: running   [==================================================>] 
3/3: running   [==================================================>] 
verify: Service converged 
```
`docker service ls`  
```
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
ex7l5v6e2pql        hungry_diffie       replicated          3/3                 alpine:latest       
```
`docker service ps hungry_diffie`  
```
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
v4q3ea3i3vwa        hungry_diffie.1     alpine:latest       node2               Running             Running about a minute ago                       
n0leadl5lbtl        hungry_diffie.2     alpine:latest       node3               Running             Running about a minute ago                       
clc8rv90kc1w        hungry_diffie.3     alpine:latest       node1               Running             Running about a minute ago       
```


### Swarm network

`docker network create --driver overlay mydrupal` 
docker@node1:~$ `docker network ls`  
```                                                                                                          
NETWORK ID          NAME                DRIVER              SCOPE
ceaea198f33e        bridge              bridge              local
fe867a30da6c        docker_gwbridge     bridge              local
6ab1a782202c        host                host                local
rsagum4kph3z        ingress             overlay             swarm
<pre>
<em>w4esb37qo7gy        mydrupal            overlay             swarm</em>
</pre>
1f68c0fd32c4        none                null                local
```


`docker service create --name psql --network mydrupal -e POSTGRES_PASSWORD=mypass postgres`  
```
k5bqo1vi8zwrjj15cyjpdpwsj
overall progress: 1 out of 1 tasks 
1/1: running   [==================================================>] 
verify: Service converged 
```
`docker service create --name drupal --network mydrupal -p 80:80 drupal`  
```
lc0sy4pnsutasg7q2lbj8gzy8
overall progress: 0 out of 1 tasks 
overall progress: 0 out of 1 tasks 
overall progress: 1 out of 1 tasks 
1/1: running   [==================================================>] 
verify: Service converged 
```
`docker service ls`  
```                                                                                                          
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
lc0sy4pnsuta        drupal              replicated          1/1                 drupal:latest       *:80->80/tcp
ex7l5v6e2pql        hungry_diffie       replicated          3/3                 alpine:latest       
k5bqo1vi8zwr        psql                replicated          1/1                 postgres:latest     
```

`docker service ps drupal`  
```                                                                                                   
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
d55rkw5bww8g        drupal.1            drupal:latest       node2               Running             Running about a minute ago
```              
docker@node1:~$ `docker service create --name search --replicas 3 -p 9200:9200 elasticsearch:2`  

utworznie w docker swarm calej aplikacji.
ilosc maszyn zmniejszona do 2. <- virtualka nie wyrabia.


logujemy sie na node1.  
tworzymy siec backend i frontend.  
`
`docker network create -d overlay backend`  
`docker network create -d overlay frontend`  

tworzymy serwis dla apliakcji do glosowania.  
`docker service create --name vote -p 80:80 --network frontend --replicas 2 bretfisher/examplevotingapp_vote`  

tworzymy serwis dla redisa:  
`docker service create --name redis --network frontend --replicas 1 redis:3.2`  

tworzymy serwis dla aplikacji worker.  
`docker service create --name worker --network frontend --network backend bretfisher/examplevotingapp_worker:java`  

tworzymy serwis dla bazy danych.  
`docker service create --name db --network backend -e POSTGRES_HOST_AUTH_METHOD=trust --mount type=volume,source=db-data,target=/var/lib/postgresql/data postgres:9.4`  

tworzymy serwis dla aplikacji wynik.  
`docker service create --name result --network backend -p 5001:80 bretfisher/examplevotingapp_result`  



### Stack

Stack jest rodzajem docker compose. Tworzymy instrukcje w yml a polecenie stack uruchamia nam serwisy w clustrze swarmowym.  
Dlaczego stack a nie compose ? Compose zostal stworzony dla zestawow developerskich aby np szybko cos sprawdzic.  
Stack zostal stworzony dla zestawo testowych/pordukcyjnch nie bedzie budowal nam obrau - pominie comendy budowania - od tego jest inny serwer np jenkins.  

`docker stack`  
```
Usage:	docker stack [OPTIONS] COMMAND

Manage Docker stacks

Options:
      --orchestrator string   Orchestrator to use (swarm|kubernetes|all)

Commands:
  deploy      Deploy a new stack or update an existing stack
  ls          List stacks
  ps          List the tasks in the stack
  rm          Remove one or more stacks
  services    List the services in the stack

Run 'docker stack COMMAND --help' for more information on a command.

```

`docker stack deploy -c nazwal_pliku.yml` 

tworzymy plik.yml  
touch stack.yml  
w srodku umieszamy zawartosc:  
```yml
version: "3"
services:

  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    environment:
      - POSTGRES_HOST_AUTH_METHOD=trust
    deploy:
      placement:
        constraints: [node.role == manager]
  vote:
    image: bretfisher/examplevotingapp_vote
    ports:
      - 5000:80
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure
  result:
    image: bretfisher/examplevotingapp_result
    ports:
      - 5001:80
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  worker:
    image: bretfisher/examplevotingapp_worker:java
    networks:
      - frontend
      - backend
    depends_on:
      - db
      - redis
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager]

  visualizer:
    image: dockersamples/visualizer
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]

networks:
  frontend:
  backend:

volumes:
  db-data:
```

Uruchaiany aplkacje z pliku stack.yml na clustrze swarm:
docker@node1:~$ `docker stack deploy -c stack.yml`   
```
voteapp                                                                                                
Creating network voteapp_default
Creating network voteapp_frontend
Creating network voteapp_backend
Creating service voteapp_worker
Creating service voteapp_visualizer
Creating service voteapp_redis
Creating service voteapp_db
Creating service voteapp_vote
Creating service voteapp_result
```
`docker service ls`  
```
ID                  NAME                 MODE                REPLICAS            IMAGE                                       PORTS
nhm0k4cw753e        voteapp_db           replicated          1/1                 postgres:9.4                                
tdjvoa5mt1rq        voteapp_redis        replicated          1/1                 redis:alpine                                *:30000->6379/tcp
4c7t0f52a6r5        voteapp_result       replicated          1/1                 bretfisher/examplevotingapp_result:latest   *:5001->80/tcp
lmxrfr88ovlw        voteapp_visualizer   replicated          1/1                 dockersamples/visualizer:latest             *:8080->8080/tcp
z1xr6o3gllp8        voteapp_vote         replicated          2/2                 bretfisher/examplevotingapp_vote:latest     *:5000->80/tcp
v5ygaih8z1c5        voteapp_worker       replicated          1/1                 bretfisher/examplevotingapp_worker:java     
```

`docker stack ls `  
```
NAME                SERVICES            ORCHESTRATOR
voteapp             6                   Swarm
```
`docker stack ps voteapp`  
```
ID                  NAME                   IMAGE                                       NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
8cwmki8qekt2        voteapp_result.1       bretfisher/examplevotingapp_result:latest   node2               Running             Running 29 minutes ago                       
dh89z4lvnfcm        voteapp_vote.1         bretfisher/examplevotingapp_vote:latest     node1               Running             Running 32 minutes ago                       
omhnhsiar1le        voteapp_db.1           postgres:9.4                                node1               Running             Running 31 minutes ago                       
z3pu0mfzxmln        voteapp_redis.1        redis:alpine                                node2               Running             Running 31 minutes ago                       
3jo6vnei6dgp        voteapp_visualizer.1   dockersamples/visualizer:latest             node2               Running             Running 27 minutes ago                       
uffrdgxst2d8        voteapp_worker.1       bretfisher/examplevotingapp_worker:java     node1               Running             Running 33 minutes ago                       
2cucguhasq80        voteapp_vote.2         bretfisher/examplevotingapp_vote:latest     node2               Running             Running 32 minutes ago
```
`docker container ls`  
```
CONTAINER ID        IMAGE                                     COMMAND                  CREATED             STATUS              PORTS               NAMES
a25dee6cedc8        postgres:9.4                              "docker-entrypoint.s…"   48 minutes ago      Up 48 minutes       5432/tcp            voteapp_db.1.omhnhsiar1le7ll04cpgj9poy
429bb60a3db3        bretfisher/examplevotingapp_vote:latest   "gunicorn app:app -b…"   50 minutes ago      Up 50 minutes       80/tcp              voteapp_vote.1.dh89z4lvnfcm1kycca7a8ck87
df6b528a89fa        bretfisher/examplevotingapp_worker:java   "java -XX:+UnlockExp…"   50 minutes ago      Up 50 minutes                           voteapp_worker.1.uffrdgxst2d8nf7v1ukl9adv0

```
`docker stack services voteapp`  
```
ID                  NAME                 MODE                REPLICAS            IMAGE                                       PORTS
4c7t0f52a6r5        voteapp_result       replicated          1/1                 bretfisher/examplevotingapp_result:latest   *:5001->80/tcp
lmxrfr88ovlw        voteapp_visualizer   replicated          1/1                 dockersamples/visualizer:latest             *:8080->8080/tcp
nhm0k4cw753e        voteapp_db           replicated          1/1                 postgres:9.4                                
tdjvoa5mt1rq        voteapp_redis        replicated          1/1                 redis:alpine                                *:30000->6379/tcp
v5ygaih8z1c5        voteapp_worker       replicated          1/1                 bretfisher/examplevotingapp_worker:java     
z1xr6o3gllp8        voteapp_vote         replicated          2/2                 bretfisher/examplevotingapp_vote:latest     *:5000->80/tcp
```
