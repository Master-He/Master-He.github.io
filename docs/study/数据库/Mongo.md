```shell
db.mycollection.find().limit(1).pretty()
db.mycollection.find({"123"}).pretty()
db.getCollection("mycollection").find().count()
db.getCollection("mycollection").find({"device.classify_id" : 70101}).count()
db.getCollection("mycollection").remove({ $and : [{"_id" : { $gte : 80000000 }}, {"_id" : { $lte : 90000000 }}] });
```

