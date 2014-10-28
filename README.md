db2-vagrant-nosql
=================

Vagrant provisioning script for IBM DB2 Express, and nosql instructions

* clone repo and cd to base dir
* download [DB2 Express-C for Linux 64-bit](http://www-01.ibm.com/software/data/db2/express-c/download.html) for the host VM
* vagrant up
* vagrant ssh
* cd /vagrant and extract db2 express archive
* exit
* vagrant provision
* vagrant ssh
* add vagrant user to db2 admin and user groups. this user will be connecting to the db
* create db
  * su
  * sudo -u db2inst1 -H bash
  * db2 create database jsondb automatic storage yes using codeset utf-8 territory us collate using system pagesize 32 K
  * db2set DB2COMM=TCPIP
  * db2 update dbm cfg using svcename 50000
  * db2stop
  * db2start
* grant db privs to vagrant
  * db2
  * connect to jsondb
  * grant dbadm, createtab, bindadd, connect, create_not_fenced, implicit_schema, load on database to user vagrant
* connect to jsondb using nosql interface, and enable the database
  * cd /opt/ibm/db2/V10.5/json/bin
  * ./db2nosql.sh -user vagrant -db jsondb
  * enable(true)
* Start the wirelistener
  * cd /opt/ibm.db2/V10.5/bin 
  * ./wplistener.sh -start -mongoPort 27017 -userid db2admin -password mypasswd -dbName jsondb 
* Connect to the database with your mongodb client

```javascript
var Db = require('mongodb').Db;
var Connection = require('mongodb').Connection
var Server = require('mongodb').Server
var format = require('util').format;

var mongo = {
        "host": "localhost",
        "port": "27017",
        "db": "mydb"
};

var db = mongo.db;
console.log("Connecting mongo at: " + mongo.host + ":" + mongo.port + ":" + mongo.db);

Db.connect(format("mongodb://%s:%s/%s?w=1", mongo.host, mongo.port, mongo.db), 
   function(err,  db) {
        db.collection('sample', function(err, collection) {
            // Start with a clean collection
            collection.remove({}, function(err, result) {
    
            // Insert a JSON document
            collection.insert({name:'Joe'},{w:1}, function(err,res){    
             collection.findOne({name:'Joe'}, function(err, item) {
              console.log("created at " + new Date(item._id.generationTime) + "\n")   

               // Insert multiple JSON documents with different schema
               for(var i = 0; i < 5; i++) {
                 collection.insert({'a':i}, {w:0});
               }
    
            });
        });
    });
    });
});
```
