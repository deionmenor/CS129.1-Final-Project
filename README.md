# CS129.1-Final-Project
Big Data Problem


## How to Load the Dataset

The dataset of Twitter tweets were obtained through a Python script using the Tweepy package. The python script outputs the twitter data of specified accounts as a JSON file.

Download the .json file into the directory you will be using.

Open the command line from that directory and run the following command:

`docker run --rm --name mongo -p 27017:27017 -p 28017:28017 -v "$(pwd)"/db:/data/db -v "$(pwd)":/data/seeds -d mongo --httpinterface --smallfiles`

Afterwards, login to the mongo container by running:

`docker exec -it mongo bash`

Run the following command to import the .json file into a collection called tweetfinal

`mongoimport --db tweetfinal --collection tweetfinal --type json --file /data/seeds/tweet.json`

There should be confirmation that the files have been imported successfully.

## How to Setup the Replicate Sets



## How to execute and run the MapReduce Functions

## How to shard the MapReduce Functions
