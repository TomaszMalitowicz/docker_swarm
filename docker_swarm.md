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
