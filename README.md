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


- ` > show dbs`

- ` > use tweetfinal`

- ` >db.tweetfinal>.find().pretty()`

3.  To verify that the data has been replicated succesfully, run the following commands.

- Login to another mongo instance

`$ mongo localhost:27018`

- Once logged-in set the current terminal as a slave

`> rs.slaveOk()`

- Then use the loaded database

`> use tweetfinal`

- Check if the data is consistent between both mongo instances

`> db.tweetfinal.find().pretty()`



## How to execute and run the MapReduce Functions

In the same mongo instance, we enter the following functions to allow us to perform MapReduce on our Data

### Map function

`map = function() {
	emit(this.created_at.charAt(11) + this.created_at.charAt(12), this.favorite_count);
}`

### Reduce function

`reduce = function(key, values) {return Array.avg(values)}`

### Runnin our MapReduce function

`results = db.runCommand({
mapReduce: 'tweetfinal',
map: map,
reduce: reduce,
query: {retweeted: false},
out: 'tweetfinal.answer'});`

### Print out the results
db.tweetfinal.answer.find().pretty();`

## How to shard the MapReduce Functions
