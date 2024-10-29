# mongo-replica-set
docker mongo replica set on the same server

create .env file in node1, node2 and node3

```bash
$ cp .env.example_node1 node1/.env
$ cp .env.example_node2 node2/.env
$ cp .env.example_node3 node3/.env
```

Create a key file
```bash
$ openssl rand -base64 741 > mongodb-keyfile
$ chmod 600 mongodb-keyfile
$ sudo chown 999 mongodb-keyfile
```

Copy key to the other nodes
```bash
$ cp mongodb-keyfile node1/key
$ cp mongodb-keyfile node2/key
$ cp mongodb-keyfile node3/key
```

Edit docker-compose.yml on node1

```bash
$ cd node1
```

change line 20

    command: mongod --keyFile /opt/keyfile/mongodb-keyfile --replSet ${REPLICA_SET_NAME} --bind_ip localhost,node1.${HOST_NAME}
to

    command: mongod

Start node1 and create users
```bash
$ docker-compose build --no-cache
$ docker-compose up
$ docker exec -it node1.cluster_mongo.local bash

root@node1:/# mongosh

> use admin
> db.createUser( {user: "userAdmin", pwd: "password", roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] });
> db.createUser( {user: "rootAdmin", pwd: "password", roles: [ { role: "root", db: "admin" } ] });
> exit;
```

Stop container and remove container by id

```bash
$ docker-compose stop
$ docker rm container-id
```

Edit docker-compose.yml on node1

change line 20

    command: mongod
to

    command: mongod --keyFile /opt/keyfile/mongodb-keyfile --replSet ${REPLICA_SET_NAME} --bind_ip localhost,node1.${HOST_NAME}

Build and start node1
```bash
$ docker-compose build --no-cache
$ docker-compose up
$ docker exec -it node1.cluster_mongo.local bash

root@node1:/# mongosh

> use admin
> db.auth("rootAdmin", "password");
```

Start node2 and node3 and return in node1

```bash
> rs.initiate({_id: "mongo_replica_set_1", members: [{_id: 0, host: "node1.cluster_mongo.local"}, {_id: 1, host: "node2.cluster_mongo.local"}, {_id: 2, host: "node3.cluster_mongo.local"}]});

```

Edit /etc/hosts and insert this line (only if the nodes reside on the same server)
```
127.0.0.1	node1.cluster_mongo.local node2.cluster_mongo.local node3.cluster_mongo.local
```
