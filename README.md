# MongoDB-Sharding-Config

- Install MongoDB
- Config Mongo Config Server
- Config Mongo Sharding Server
- Config Mongo Routing Server
- Mongo sharding to Keyfile Authentication

### Install MongoDB
1. Update the packages

	`$ sudo apt update`

2. Install the MongoDB

	`$ sudo apt install -y mongodb`

3. Check the service status

	`$ sudo systemctl status mongodb`

### Config Mongo Config Server

1. Create a separate database for the config server.

	`$ mkdir /data/configdb`

2. Start the mongodb instance in configuration mode

	`$  mongod --configsvr --replSet ctf --bind_ip 0.0.0.0 --port 27017 --dbpath /data/configdb/`

3. add config server

```javascript
    rs.initiate(
  {
    _id: "ctf",
    configsvr: true,
    members: [
      { _id : 0, host : "192.168.16.122:27017" },
      { _id : 1, host : "192.168.16.123:27017" }
    ]
  }
)
```
    
	

### Config Mongo Sharding Server

1. Create a separate database for the Sharding server.

	`$ mkdir /data/db`

2. Start the mongodb instance in configuration mode

	`$ mongod --shardsvr --replSet shard1rs --bind_ip 0.0.0.0 --port 27017 --dbpath /data/db/`

3. add shard server

```javascript
    rs.initiate(
  {
    _id: "shard1rs",
    members: [
      { _id : 0, host : "192.168.16.127:27017" },
      { _id : 1, host : "192.168.16.128:27017" }
    ]
  }
)
```

### Config Mongo Routing Server

1. Start the mongos instance by specifying the configuration server.

	`$ mongos --configdb ctf/192.168.16.122:27017,192.168.16.123:27017 --bind_ip 192.168.16.135 --port 27017`

2. add shard.

	`sh.addShard("shard1rs/192.168.16.127:27017,192.168.16.128:27017")`

### Mongo sharding to Keyfile Authentication

- Generate a keyfile

1. Generate a keyfile.

	`$ openssl rand -base64 756 > <path-to-keyfile>`

2. Change file permissions.

	`$ chmod 400 <path-to-keyfile>`

- Config server edit the config file

1. Config file.

	`$ sudo nano /etc/mongos.conf`
 
2. Edit the config file.
    
```javascript
   security:
         authorization: "enabled"
```

- Config Mongo Config Server

	`$ sudo mongod --configsvr --replSet ctf --bind_ip 192.168.16.122 --port 27017 --dbpath /data/configdb/ -f /etc/mongos.conf --keyFile <path-to-keyfile>`

- Config Mongo Sharding Server

	`$ sudo mongod --shardsvr --replSet shard1rs --bind_ip 192.168.16.127 --port 27017 --dbpath /data/db/ -f /etc/mongos.conf --keyFile <path-to-keyfile>`
    
- Config Mongo Routing Server

	`$ sudo mongos --configdb ctf/192.168.16.122:27017,192.168.16.123:27017 --bind_ip 192.168.16.135 --port 27017 --keyFile <path-to-keyfile>`
