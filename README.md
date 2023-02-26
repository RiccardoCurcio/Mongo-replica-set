# mongo-replica-set
docker mongo replica set

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

    command: mongod --keyFile /opt/keyfile/mongodb-keyfile --replSet "rs0"
to

    command: mongod

Start node1 and create users
```bash
$ docker-compose build --no-cache
$ docker-compose up
$ docker exec -it node1.clustermongo.local bash

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

    command: mongod --keyFile /opt/keyfile/mongodb-keyfile --replSet "rs0"

Build and start node1
```bash
$ docker-compose build --no-cache
$ docker-compose up
$ docker exec -it node1.clustermongo.local bash

root@node1:/# mongosh

> use admin
> db.auth("rootAdmin", "password");
> rs.initiate()
```

Start node2 and node3 and return in node1

```bash
> rs.add("node2.clustermongo.local")
> rs.add("node2.clustermongo.local")
```

Edit /etc/hosts and insert this line (only if the nodes reside on the same server)
```
127.0.0.1	node1.clustermongo.local node2.clustermongo.local node3.clustermongo.local
```
