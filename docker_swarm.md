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