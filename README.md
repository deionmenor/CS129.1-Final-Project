# CS129.1-Final-Project
Big Data Problem


## How to Setup the Replicate Sets

1. Create a folder for our replicate set
2. Create three folders under replicate named mongo1, mongo2, mongo3

`mkdir mongo1 mongo2 mongo3`

3. To run the servers, run each of the following lines in different terminals

`mongod --replSet replicate --dbpath mongo1 --port 27017 --rest`

`mongod --replSet replicate --dbpath mongo1 --port 27018 --rest`

`mongod --replSet replicate --dbpath mongo1 --port 27019 --rest`

4. Add a config file to one of the mongo instances, namely the one connected to port 27017

`mongo localhost:27017`

`var config = {
  "\_id": "replicate",
  "version" : 1,
  "members" :   [
    {
      "\_id" : 0,
      "host" : "localhost:27017",
      "priority" : 1
    },
    {
      "\_id" : 1,
      "host" : "localhost:27018",
      "priority" : 0
    },
    {
      "\_id" : 2,
      "host" : "localhost:27019",
      "priority" : 0
    }
  ]
}`

5. Initialize the replica set using the config variable

`rs.initiate(config)`

## How to Load the Dataset into the Replicate Setup


1.  Load the dataset into the mongo instance using the following code.

  `mongoimport --db tweetfinal --collection tweetfinal --type json --file ADMUSanggu.json -h localhost:27017`
  
  `mongoimport --db tweetfinal --collection tweetfinal --type json --file GetBlued.json -h localhost:27017`
  
  `mongoimport --db tweetfinal --collection tweetfinal --type json --file ateneodemanilau.json -h localhost:27017`
  
  `mongoimport --db tweetfinal --collection tweetfinal --type json --file TheGUIDON.json -h localhost:27017`
  
  `mongoimport --db tweetfinal --collection tweetfinal --type json --file TheGUIDONSports.json -h localhost:27017`

2. To find out if the files are loaded succesfully into the database, we run the following commands


`show dbs`

`use tweetfinal`

`db.tweetfinal>.find().pretty()`

3.  To verify that the data has been replicated succesfully, run the following commands.

- Login to another mongo instance

`mongo localhost:27018`

- Once logged-in set the current terminal as a slave

`rs.slaveOk()`

- Then use the loaded database

`use tweetfinal`

- Check if the data is consistent between both mongo instances

`db.tweetfinal.find().pretty()`



## How to execute and run the MapReduce Functions

In the same mongo instance, we enter the following functions to allow us to perform MapReduce on our Data

#### Map function

`map = function() {
	emit(this.created_at.charAt(11) + this.created_at.charAt(12), this.favorite_count);
}`

#### Reduce function

`reduce = function(key, values) {return Array.avg(values)}`

#### Running our MapReduce function

`results = db.runCommand({
mapReduce: 'tweetfinal',
map: map,
reduce: reduce,
query: {retweeted: false},
out: 'tweetfinal.answer'});`

#### Print out the results
`db.tweetfinal.answer.find().pretty();`

## Sharding

- Make another folder called `sharding` with subfolders config1, config2, node1, and node2

`mkdir sharding`

`cd sharding`

`mkdir config1 config2 node1 node2`

- Download and place the `config.yml`, `shard.yml`, and `mongos.yml` inside the `sharding` folder

### Setting up config server replica set

- Create the config server replica set by running each line below in two separate terminals

`mongod --config config.yml --bind_ip 127.0.0.1 --port 27010 --dbpath config1`

`mongod --config config.yml --bind_ip 127.0.0.1 --port 27011 --dbpath config2`

- In another terminal, enter the command `mongo --host 127.0.0.1 --port 27011` to connect to one of the config servers

- Initiate the replica set by running the following into the mongo shell

`rs.initiate({
	_id: "config",
	configsvr: true,
	members: [
		{ _id: 0, host: "127.0.0.1:27010", priority: 1 },
		{ _id: 1, host: "127.0.0.1:27011", priority: 0 }
	]
})`

### Setting up shard replica set

- Create the shard replica sets via the command line by inputting the following on two separate terminals as well

`mongod --config shard.yml --bind_ip 127.0.0.1 --port 27012 --dbpath node1`

`mongod --config shard.yml --bind_ip 127.0.0.1 --port 27013 --dbpath node2`

- In another terminal, enter the command `mongo --host 127.0.0.1 --port 27012` to connect to one of the replica set members

- Initiate the shard replica set by running the following code in the shell

`rs.initiate({
	_id: 'shard',
	members: [
		{ _id: 0, host: "127.0.0.1:27012", priority: 1 },
		{ _id: 1, host: "127.0.0.1:27013", priority: 0 }
	]
})`

### Setting up mongos 

- Connect a mongos to the cluster by entering `mongos --config mongos.yml --bind_ip 127.0.0.1 --port 27014` into another terminal

- In another terminal, enter the command `mongo --host 127.0.0.1 --port 27014` to connect to the mongos


### Adding shards to cluster

- In the shell, run the following lines in order to add node1 and node2 as shards

`sh.addShard( "shard/127.0.0.1:27012")`

`sh.addShard( "shard/127.0.0.1:27013")`

- Type `sh.enableSharding("tweetfinal")` to enable sharding in our tweetfinal database

### Create a sharded collection

- Type `use tweetfinal` and enter the following commands to create an empty collection `tweets` to be used for hashed sharding

`db.createCollection('tweets')`

`db.tweets.createIndex({id:1}, {unique: true})`

`sh.shardCollection("tweetfinal.tweets", { _id : "hashed" } )`

### Importing of data

- Exit the shell and run the following commands to import our 5 JSON files

`mongoimport --db tweetfinal --collection tweets  --type json --host "127.0.0.1:27014" --file "ADMUSanggu.json"`

`mongoimport --db tweetfinal --collection tweets  --type json --host "127.0.0.1:27014" --file "ateneodemanilau.json"`

`mongoimport --db tweetfinal --collection tweets  --type json --host "127.0.0.1:27014" --file "TheGUIDON.json"`

`mongoimport --db tweetfinal --collection tweets  --type json --host "127.0.0.1:27014" --file "TheGUIDONSports.json"`

`mongoimport --db tweetfinal --collection tweets  --type json --host "127.0.0.1:27014" --file "GetBlued.json"`

- MapReduce functions can be tested out similarly to the steps above for replicate sets








